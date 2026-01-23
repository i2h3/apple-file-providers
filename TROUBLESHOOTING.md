# Troubleshooting Guide

Common problems and their solutions when working with Apple's File Provider framework.

## Table of Contents

- [Extension Not Loading](#extension-not-loading)
- [Files Not Appearing](#files-not-appearing)
- [Enumeration Issues](#enumeration-issues)
- [Upload/Download Problems](#uploaddownload-problems)
- [Performance Issues](#performance-issues)
- [Crash Scenarios](#crash-scenarios)
- [Debugging Tips](#debugging-tips)

## Extension Not Loading

### Problem: Extension doesn't appear in Files.app

**Possible Causes:**
1. Domain not registered
2. Extension not properly configured in Info.plist
3. System hasn't discovered the extension
4. Code signing issues

**Solutions:**

1. **Verify domain registration:**
```swift
NSFileProviderManager.getDomainsWithCompletionHandler { domains, error in
    print("Registered domains: \(domains)")
    if domains.isEmpty {
        // Register your domain
        setupFileProvider()
    }
}
```

2. **Check Info.plist:**
Ensure these keys are present in the extension's Info.plist:
```xml
<key>NSExtensionFileProviderSupportsEnumeration</key>
<true/>
<key>NSExtensionPointIdentifier</key>
<string>com.apple.fileprovider-nonui</string>
```

3. **Force system to rediscover:**
- Delete and reinstall the app
- Restart the device
- On macOS: Kill fileproviderd process

4. **Check code signing:**
- Ensure extension and app have same team ID
- Verify provisioning profiles are valid
- Check App Groups entitlement is enabled for both targets

### Problem: Domain registered but not visible in sidebar (macOS)

**Possible Causes:**
1. Domain display name is empty or invalid
2. Extension crashes immediately on load
3. System cache issue

**Solutions:**

1. **Set proper display name:**
```swift
let domain = NSFileProviderDomain(
    identifier: NSFileProviderDomainIdentifier(rawValue: "com.myapp.provider"),
    displayName: "My Cloud" // Must not be empty
)
```

2. **Check Console.app for crashes:**
Filter by process `fileproviderd` and look for crash reports or error messages.

3. **Reset domain:**
```swift
// Remove and re-add the domain
NSFileProviderManager.remove(domainIdentifier) { error in
    // Then add it again
    NSFileProviderManager.add(domain) { error in
        print("Domain re-added")
    }
}
```

## Files Not Appearing

### Problem: Files don't show up in Files.app/Finder

**Possible Causes:**
1. Enumerator not returning items
2. Items have invalid parent hierarchy
3. Items returned but not materialized
4. Caching issue

**Solutions:**

1. **Check enumerator is called:**
```swift
override func enumerator(for containerItemIdentifier: NSFileProviderItemIdentifier) throws -> NSFileProviderEnumerator {
    os_log(.debug, log: .enumeration, "Creating enumerator for: %{public}@", containerItemIdentifier.rawValue)
    return MyEnumerator(identifier: containerItemIdentifier)
}
```

2. **Verify items are enumerated:**
```swift
func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    let items = database.items(in: containerIdentifier)
    os_log(.info, log: .enumeration, "Enumerating %d items", items.count)
    
    observer.didEnumerate(items)
    observer.finishEnumerating(upTo: nil)
}
```

3. **Validate parent hierarchy:**
```swift
// All items must have valid parent chain to root
func validateItem(_ item: FileProviderItem) -> Bool {
    var currentIdentifier = item.parentItemIdentifier
    
    while currentIdentifier != .rootContainer {
        guard let parent = database.item(for: currentIdentifier) else {
            os_log(.error, log: .fileProvider, "Orphaned item: %{public}@", item.filename)
            return false
        }
        currentIdentifier = parent.parentItemIdentifier
    }
    
    return true
}
```

4. **Force refresh:**
```swift
// Signal enumerator to refresh
manager.signalEnumerator(for: .rootContainer) { error in
    if let error = error {
        os_log(.error, log: .fileProvider, "Signal failed: %{public}@", error.localizedDescription)
    }
}
```

### Problem: Files appear but show as zero bytes

**Possible Causes:**
1. documentSize not set correctly
2. Item not materialized
3. Placeholder not created

**Solutions:**

1. **Set correct document size:**
```swift
let item = FileProviderItem(
    // ...
    documentSize: NSNumber(value: actualFileSize), // Must be accurate
    // ...
)
```

2. **Create placeholders:**
```swift
// In extension
override func providePlaceholder(at url: URL, completionHandler: @escaping (Error?) -> Void) {
    let identifier = manager.persistentIdentifier(for: url)
    
    guard let item = try? item(for: identifier) else {
        completionHandler(NSFileProviderError(.noSuchItem))
        return
    }
    
    do {
        try NSFileProviderManager.writePlaceholder(at: url, withMetadata: item)
        completionHandler(nil)
    } catch {
        completionHandler(error)
    }
}
```

## Enumeration Issues

### Problem: Infinite enumeration loop (100% CPU usage)

**Possible Causes:**
1. Never calling finishEnumerating
2. Returning same page repeatedly
3. Invalid sync anchor

**Solutions:**

1. **Always call completion:**
```swift
func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    do {
        let items = try fetchItems(page: page)
        observer.didEnumerate(items)
        
        let nextPage = items.isEmpty ? nil : NSFileProviderPage("\(pageNumber + 1)".data(using: .utf8)!)
        observer.finishEnumerating(upTo: nextPage)
    } catch {
        observer.finishEnumeratingWithError(error)
    }
}
```

2. **Implement proper pagination:**
```swift
func fetchItems(page: NSFileProviderPage) -> [FileProviderItem] {
    let pageNumber = Int(String(data: page.rawValue, encoding: .utf8) ?? "0") ?? 0
    let offset = pageNumber * pageSize
    
    return database.items(limit: pageSize, offset: offset)
}
```

3. **Use unique sync anchors:**
```swift
func enumerateChanges(for observer: NSFileProviderChangeObserver, from anchor: NSFileProviderSyncAnchor) {
    let timestamp = String(data: anchor.rawValue, encoding: .utf8) ?? "0"
    let changes = database.changesSince(timestamp: timestamp)
    
    observer.didUpdate(changes.updated)
    observer.didDeleteItems(withIdentifiers: changes.deleted)
    
    // Create new unique anchor
    let newAnchor = NSFileProviderSyncAnchor("\(Date().timeIntervalSince1970)".data(using: .utf8)!)
    observer.finishEnumeratingChanges(upTo: newAnchor, moreComing: false)
}
```

### Problem: Files appear and disappear randomly

**Possible Causes:**
1. Inconsistent enumeration results
2. Race condition in database
3. Sync anchor issues

**Solutions:**

1. **Ensure consistent queries:**
```swift
// Use transactions for consistency
func items(in container: NSFileProviderItemIdentifier) -> [FileProviderItem] {
    return database.transaction {
        return database.fetch(parentIdentifier: container)
    }
}
```

2. **Thread-safe database access:**
```swift
class DatabaseManager {
    private let queue = DispatchQueue(label: "com.myapp.database", attributes: .concurrent)
    
    func items(in container: NSFileProviderItemIdentifier) -> [FileProviderItem] {
        return queue.sync {
            // Read from database
        }
    }
    
    func save(_ item: FileProviderItem) {
        queue.async(flags: .barrier) {
            // Write to database
        }
    }
}
```

## Upload/Download Problems

### Problem: Downloads fail silently

**Possible Causes:**
1. Network error not handled
2. File not written to expected URL
3. Progress not configured

**Solutions:**

1. **Proper error handling:**
```swift
override func fetchContents(
    for itemIdentifier: NSFileProviderItemIdentifier,
    version: NSFileProviderItemVersion?,
    request: NSFileProviderRequest,
    completionHandler: @escaping (URL?, NSFileProviderItem?, Error?) -> Void
) -> Progress {
    let progress = Progress(totalUnitCount: 100)
    
    Task {
        do {
            let url = try await download(itemIdentifier, progress: progress)
            let item = try self.item(for: itemIdentifier)
            completionHandler(url, item, nil)
        } catch let error as NSError {
            os_log(.error, log: .network, "Download failed: %{public}@", error.localizedDescription)
            
            // Convert to FileProvider error if needed
            let fpError: Error
            if error.domain == NSURLErrorDomain {
                fpError = NSFileProviderError(.serverUnreachable)
            } else {
                fpError = error
            }
            
            completionHandler(nil, nil, fpError)
        }
    }
    
    return progress
}
```

2. **Write to correct URL:**
```swift
func download(_ identifier: NSFileProviderItemIdentifier, progress: Progress) async throws -> URL {
    let tempURL = FileManager.default.temporaryDirectory.appendingPathComponent(UUID().uuidString)
    
    // Download to temp location
    try await networkClient.download(identifier: identifier, to: tempURL, progress: progress)
    
    // Move to final location
    let finalURL = storage.url(for: identifier)
    try? FileManager.default.removeItem(at: finalURL)
    try FileManager.default.moveItem(at: tempURL, to: finalURL)
    
    return finalURL
}
```

### Problem: Uploads don't trigger

**Possible Causes:**
1. modifyItem not implemented
2. Item capabilities don't allow writing
3. File coordinator not used

**Solutions:**

1. **Implement modifyItem:**
```swift
override func modifyItem(
    _ item: NSFileProviderItem,
    baseVersion version: NSFileProviderItemVersion,
    changedFields: NSFileProviderItemFields,
    contents newContents: URL?,
    options: NSFileProviderModifyItemOptions,
    request: NSFileProviderRequest,
    completionHandler: @escaping (NSFileProviderItem?, NSFileProviderItemFields, Bool, Error?) -> Void
) -> Progress {
    // Handle upload
    return uploadItem(item, contents: newContents, completionHandler: completionHandler)
}
```

2. **Set proper capabilities:**
```swift
let item = FileProviderItem(
    // ...
    capabilities: [
        .allowsReading,
        .allowsWriting,     // Required for uploads
        .allowsRenaming,
        .allowsDeleting
    ],
    // ...
)
```

### Problem: Large file uploads fail or timeout

**Possible Causes:**
1. Not using background URLSession
2. Extension terminated during upload
3. Memory issues

**Solutions:**

1. **Use background URLSession:**
```swift
class UploadManager {
    let session: URLSession
    
    init() {
        let config = URLSessionConfiguration.background(
            withIdentifier: "com.myapp.fileupload"
        )
        config.isDiscretionary = false
        config.sessionSendsLaunchEvents = true
        
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func upload(_ fileURL: URL, to identifier: NSFileProviderItemIdentifier) {
        var request = URLRequest(url: serverURL)
        request.httpMethod = "POST"
        
        let task = session.uploadTask(with: request, fromFile: fileURL)
        
        // Store mapping for completion handler
        taskMap[task.taskIdentifier] = identifier
        
        task.resume()
    }
}
```

2. **Handle background completion:**
```swift
// In AppDelegate
func application(
    _ application: UIApplication,
    handleEventsForBackgroundURLSession identifier: String,
    completionHandler: @escaping () -> Void
) {
    // Store completion handler
    backgroundCompletionHandler = completionHandler
}

// In URLSessionDelegate
func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
    DispatchQueue.main.async {
        self.backgroundCompletionHandler?()
        self.backgroundCompletionHandler = nil
    }
}
```

## Performance Issues

### Problem: Slow enumeration

**Possible Causes:**
1. Loading too many items at once
2. Slow database queries
3. Thumbnail generation blocking
4. Network calls in enumeration

**Solutions:**

1. **Implement pagination:**
```swift
let pageSize = 100

func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    let offset = pageOffset(from: page)
    let items = database.items(limit: pageSize, offset: offset)
    
    observer.didEnumerate(items)
    
    let nextPage = items.count == pageSize ? createPage(offset: offset + pageSize) : nil
    observer.finishEnumerating(upTo: nextPage)
}
```

2. **Optimize database:**
```sql
-- Add indexes for common queries
CREATE INDEX idx_parent ON items(parent_identifier);
CREATE INDEX idx_modified ON items(modified_date);
```

3. **Generate thumbnails asynchronously:**
```swift
struct FileProviderItem: NSFileProviderItem {
    var thumbnailData: Data? {
        // Return cached thumbnail if available
        return thumbnailCache[identifier]
    }
}

// Generate in background
func generateThumbnails(for items: [FileProviderItem]) {
    DispatchQueue.global(qos: .utility).async {
        for item in items {
            if let thumbnail = generateThumbnail(for: item) {
                thumbnailCache[item.identifier] = thumbnail
            }
        }
    }
}
```

4. **Never make network calls in enumeration:**
```swift
// ❌ DON'T DO THIS
func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    // This blocks enumeration!
    let items = networkClient.fetchItems() // WRONG
    observer.didEnumerate(items)
}

// ✅ DO THIS
func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    // Enumerate from local cache
    let items = database.items()
    observer.didEnumerate(items)
    
    // Sync in background
    syncInBackground()
}
```

### Problem: High memory usage

**Possible Causes:**
1. Loading all items into memory
2. Caching too much data
3. Memory leaks

**Solutions:**

1. **Use database cursors:**
```swift
func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    autoreleasepool {
        let items = database.items(page: page)
        observer.didEnumerate(items)
        // Items released here
    }
    observer.finishEnumerating(upTo: nextPage)
}
```

2. **Limit cache size:**
```swift
class ThumbnailCache {
    private var cache: [NSFileProviderItemIdentifier: Data] = [:]
    private let maxSize = 100
    
    func set(_ thumbnail: Data, for identifier: NSFileProviderItemIdentifier) {
        if cache.count >= maxSize {
            // Remove oldest
            cache.removeValue(forKey: cache.keys.first!)
        }
        cache[identifier] = thumbnail
    }
}
```

3. **Profile with Instruments:**
- Use Leaks instrument to find memory leaks
- Use Allocations to track memory usage
- Look for strong reference cycles

## Crash Scenarios

### Problem: Crash with "Illegal seek"

**Cause:** Trying to access file at URL that doesn't exist

**Solution:**
```swift
override func startProvidingItem(at url: URL, completionHandler: @escaping (Error?) -> Void) {
    // Ensure file exists at URL
    guard !FileManager.default.fileExists(atPath: url.path) else {
        completionHandler(nil)
        return
    }
    
    // Materialize the file
    let identifier = persistentIdentifier(for: url)
    fetchContents(for: identifier) { downloadedURL, _, error in
        if let error = error {
            completionHandler(error)
            return
        }
        
        if let downloadedURL = downloadedURL {
            do {
                try FileManager.default.copyItem(at: downloadedURL, to: url)
                completionHandler(nil)
            } catch {
                completionHandler(error)
            }
        }
    }
}
```

### Problem: Crash in enumeration

**Cause:** Invalid item hierarchy or nil values

**Solution:**
```swift
func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
    do {
        let items = try database.items(in: containerIdentifier)
        
        // Validate items before returning
        let validItems = items.filter { item in
            guard item.parentItemIdentifier != item.itemIdentifier else {
                os_log(.error, log: .enumeration, "Item is its own parent: %{public}@", item.filename)
                return false
            }
            return true
        }
        
        observer.didEnumerate(validItems)
        observer.finishEnumerating(upTo: nil)
    } catch {
        os_log(.error, log: .enumeration, "Enumeration failed: %{public}@", error.localizedDescription)
        observer.finishEnumeratingWithError(error)
    }
}
```

## Debugging Tips

### View Extension Logs

**Console.app (macOS):**
1. Open Console.app
2. Filter by process: `fileproviderd`
3. Filter by subsystem: your bundle identifier
4. Use predicates: `subsystem == "com.myapp.FileProvider" AND category == "Enumeration"`

**Xcode Console:**
1. Debug > Attach to Process > fileproviderd
2. Or create a scheme that launches Files.app

### Common Log Predicates

```
# All errors
messageType == error

# Specific subsystem
subsystem == "com.myapp.FileProvider"

# Specific category
category == "Enumeration"

# Combined
subsystem == "com.myapp.FileProvider" AND category == "Network" AND messageType >= info
```

### Attach Debugger

**Xcode:**
1. Debug > Attach to Process by PID or Name...
2. Enter: `fileproviderd`
3. Set breakpoints in extension code

**LLDB:**
```bash
# Find process
ps aux | grep fileproviderd

# Attach
lldb -p <pid>

# Set breakpoint
breakpoint set --name "-[MyFileProviderExtension enumeratorForContainerItemIdentifier:]"
```

### Inspect Database

If using SQLite:
```bash
# Find database location
# Usually in: ~/Library/Group Containers/<app-group>/
cd ~/Library/Group\ Containers/<app-group-id>/

# Open with sqlite3
sqlite3 fileprovider.db

# Inspect schema
.schema

# Query items
SELECT * FROM items LIMIT 10;
```

### Force Extension Reload

**macOS:**
```bash
# Kill fileproviderd process
killall fileproviderd

# System will restart it on next access
```

**iOS:**
```bash
# Restart device or
# Delete and reinstall app
```

### Test Network Issues

**Use Charles Proxy or similar:**
1. Configure device/simulator to use proxy
2. Throttle bandwidth
3. Introduce delays
4. Simulate failures
5. Inspect requests/responses

**Simulate Airplane Mode:**
```swift
#if DEBUG
if UserDefaults.standard.bool(forKey: "SimulateOffline") {
    throw NSError(domain: NSURLErrorDomain, code: NSURLErrorNotConnectedToInternet)
}
#endif
```

### Extension Logging Best Practices

```swift
import os.log

extension OSLog {
    static let fileProvider = OSLog(subsystem: "com.myapp", category: "FileProvider")
}

// Use appropriate levels
os_log(.debug, log: .fileProvider, "Method called")           // Verbose, development only
os_log(.info, log: .fileProvider, "Operation completed")      // Informational
os_log(.default, log: .fileProvider, "State change")          // Default level
os_log(.error, log: .fileProvider, "Operation failed")        // Errors
os_log(.fault, log: .fileProvider, "Critical failure")        // Critical issues

// Include context
os_log(.info, log: .fileProvider, "Enumerated %d items in %{public}@", count, containerName)

// Mark sensitive data
os_log(.debug, log: .fileProvider, "Auth token: %{private}@", token)
```
