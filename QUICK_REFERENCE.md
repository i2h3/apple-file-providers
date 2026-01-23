# Quick Reference Guide

This guide provides quick code snippets and examples for common File Provider implementation patterns.

## Table of Contents

- [Basic Setup](#basic-setup)
- [Domain Management](#domain-management)
- [Enumerator Implementation](#enumerator-implementation)
- [Item Implementation](#item-implementation)
- [Content Management](#content-management)
- [Error Handling](#error-handling)
- [Logging](#logging)

## Basic Setup

### Info.plist Configuration

```xml
<key>NSExtension</key>
<dict>
    <key>NSExtensionFileProviderSupportsEnumeration</key>
    <true/>
    <key>NSExtensionPointIdentifier</key>
    <string>com.apple.fileprovider-nonui</string>
    <key>NSExtensionPrincipalClass</key>
    <string>$(PRODUCT_MODULE_NAME).FileProviderExtension</string>
</dict>
```

### Extension Class

```swift
import FileProvider

class FileProviderExtension: NSFileProviderExtension {
    
    override init() {
        super.init()
        // Initialize your storage, database, etc.
    }
    
    override func item(for identifier: NSFileProviderItemIdentifier) throws -> NSFileProviderItem {
        // Return the item for the given identifier
        guard let item = database.item(for: identifier) else {
            throw NSFileProviderError(.noSuchItem)
        }
        return item
    }
    
    override func enumerator(for containerItemIdentifier: NSFileProviderItemIdentifier) throws -> NSFileProviderEnumerator {
        // Return an enumerator for the container
        return MyEnumerator(identifier: containerItemIdentifier)
    }
}
```

## Domain Management

### Adding a Domain (in Main App)

```swift
import FileProvider

func setupFileProvider() {
    let domain = NSFileProviderDomain(
        identifier: NSFileProviderDomainIdentifier(rawValue: "com.myapp.fileprovider"),
        displayName: "My Cloud Storage"
    )
    
    NSFileProviderManager.add(domain) { error in
        if let error = error {
            print("Failed to add domain: \(error)")
        } else {
            print("Domain added successfully")
        }
    }
}
```

### Removing a Domain

```swift
func removeFileProvider() {
    let domainIdentifier = NSFileProviderDomainIdentifier(rawValue: "com.myapp.fileprovider")
    
    NSFileProviderManager.remove(domainIdentifier) { error in
        if let error = error {
            print("Failed to remove domain: \(error)")
        } else {
            print("Domain removed successfully")
        }
    }
}
```

### Getting Manager for Domain

```swift
let domainIdentifier = NSFileProviderDomainIdentifier(rawValue: "com.myapp.fileprovider")
guard let manager = NSFileProviderManager(for: domainIdentifier) else {
    fatalError("No manager for domain")
}
```

## Enumerator Implementation

### Basic Enumerator

```swift
import FileProvider

class MyEnumerator: NSObject, NSFileProviderEnumerator {
    let containerIdentifier: NSFileProviderItemIdentifier
    
    init(identifier: NSFileProviderItemIdentifier) {
        self.containerIdentifier = identifier
        super.init()
    }
    
    func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
        // Fetch items from your storage
        let items = database.items(in: containerIdentifier, startingAt: page)
        
        // Send items to observer
        observer.didEnumerate(items)
        
        // Signal completion
        observer.finishEnumerating(upTo: nextPage)
    }
    
    func enumerateChanges(for observer: NSFileProviderChangeObserver, from anchor: NSFileProviderSyncAnchor) {
        // Get changes since anchor
        let changes = database.changes(since: anchor)
        
        // Report updated items
        observer.didUpdate(changes.updated)
        
        // Report deleted items
        observer.didDeleteItems(withIdentifiers: changes.deleted)
        
        // Create new anchor
        let newAnchor = NSFileProviderSyncAnchor(Date().timeIntervalSince1970.description.data(using: .utf8)!)
        
        // Signal completion
        observer.finishEnumeratingChanges(upTo: newAnchor, moreComing: false)
    }
    
    func currentSyncAnchor(completionHandler: @escaping (NSFileProviderSyncAnchor?) -> Void) {
        let anchor = NSFileProviderSyncAnchor(database.currentAnchor.data(using: .utf8)!)
        completionHandler(anchor)
    }
}
```

### Working Set Enumerator

```swift
class WorkingSetEnumerator: NSObject, NSFileProviderEnumerator {
    
    func enumerateItems(for observer: NSFileProviderEnumerationObserver, startingAt page: NSFileProviderPage) {
        // Return only recently changed items
        let recentlyChanged = database.itemsModifiedInLast(days: 7)
        
        observer.didEnumerate(recentlyChanged)
        observer.finishEnumerating(upTo: nil)
    }
    
    func enumerateChanges(for observer: NSFileProviderChangeObserver, from anchor: NSFileProviderSyncAnchor) {
        // For working set, enumerate all recent changes
        let changes = database.recentChanges(since: anchor)
        
        observer.didUpdate(changes.updated)
        observer.didDeleteItems(withIdentifiers: changes.deleted)
        
        let newAnchor = NSFileProviderSyncAnchor(Date().timeIntervalSince1970.description.data(using: .utf8)!)
        observer.finishEnumeratingChanges(upTo: newAnchor, moreComing: false)
    }
}
```

## Item Implementation

### NSFileProviderItem Struct

```swift
import FileProvider
import UniformTypeIdentifiers

struct FileProviderItem: NSFileProviderItem {
    let identifier: NSFileProviderItemIdentifier
    let parentItemIdentifier: NSFileProviderItemIdentifier
    let filename: String
    let typeIdentifier: String
    let capabilities: NSFileProviderItemCapabilities
    let documentSize: NSNumber?
    let contentModificationDate: Date?
    let creationDate: Date?
    let lastUsedDate: Date?
    let isMaterialized: Bool
    
    var itemIdentifier: NSFileProviderItemIdentifier { identifier }
    
    var itemVersion: NSFileProviderItemVersion {
        // Version should change when content or metadata changes
        let versionData = "\(contentModificationDate?.timeIntervalSince1970 ?? 0)".data(using: .utf8)!
        return NSFileProviderItemVersion(contentVersion: versionData, metadataVersion: versionData)
    }
    
    var contentType: UTType {
        UTType(typeIdentifier) ?? .data
    }
    
    // Optional: Thumbnail
    var thumbnailData: Data? {
        // Return thumbnail image data if available
        return nil
    }
    
    // Optional: Tags
    var tagData: Data? {
        return nil
    }
    
    // For folders
    var childItemCount: NSNumber? {
        if contentType == .folder {
            return NSNumber(value: database.childCount(for: identifier))
        }
        return nil
    }
}
```

### Creating Items

```swift
// File item
let fileItem = FileProviderItem(
    identifier: NSFileProviderItemIdentifier(rawValue: "file123"),
    parentItemIdentifier: .rootContainer,
    filename: "document.pdf",
    typeIdentifier: UTType.pdf.identifier,
    capabilities: [.allowsReading, .allowsWriting, .allowsRenaming, .allowsDeleting],
    documentSize: NSNumber(value: 1024000),
    contentModificationDate: Date(),
    creationDate: Date(),
    lastUsedDate: nil,
    isMaterialized: true
)

// Folder item
let folderItem = FileProviderItem(
    identifier: NSFileProviderItemIdentifier(rawValue: "folder456"),
    parentItemIdentifier: .rootContainer,
    filename: "Documents",
    typeIdentifier: UTType.folder.identifier,
    capabilities: [.allowsAddingSubItems, .allowsContentEnumerating, .allowsDeleting, .allowsRenaming],
    documentSize: nil,
    contentModificationDate: Date(),
    creationDate: Date(),
    lastUsedDate: nil,
    isMaterialized: false
)
```

## Content Management

### Fetching Contents

```swift
override func fetchContents(
    for itemIdentifier: NSFileProviderItemIdentifier,
    version requestedVersion: NSFileProviderItemVersion?,
    request: NSFileProviderRequest,
    completionHandler: @escaping (URL?, NSFileProviderItem?, Error?) -> Void
) -> Progress {
    
    let progress = Progress(totalUnitCount: 100)
    
    // Check if already materialized
    if let localURL = storage.localURL(for: itemIdentifier), FileManager.default.fileExists(atPath: localURL.path) {
        completionHandler(localURL, nil, nil)
        return progress
    }
    
    // Download from server
    Task {
        do {
            let downloadedURL = try await networkClient.download(itemIdentifier: itemIdentifier, progress: progress)
            let item = try self.item(for: itemIdentifier)
            completionHandler(downloadedURL, item, nil)
        } catch {
            completionHandler(nil, nil, error)
        }
    }
    
    return progress
}
```

### Providing Item

```swift
override func startProvidingItem(
    at url: URL,
    completionHandler: @escaping (Error?) -> Void
) {
    // The file should already be at the URL from fetchContents
    // If not, materialize it now
    
    guard FileManager.default.fileExists(atPath: url.path) else {
        completionHandler(NSFileProviderError(.noSuchItem))
        return
    }
    
    completionHandler(nil)
}
```

### Importing Document

```swift
override func importDocument(
    at fileURL: URL,
    toParentItemIdentifier parentItemIdentifier: NSFileProviderItemIdentifier,
    completionHandler: @escaping (NSFileProviderItem?, Error?) -> Void
) -> Progress {
    
    let progress = Progress(totalUnitCount: 100)
    
    Task {
        do {
            // Generate new identifier
            let itemIdentifier = NSFileProviderItemIdentifier(rawValue: UUID().uuidString)
            
            // Copy to storage
            let destinationURL = storage.url(for: itemIdentifier)
            try FileManager.default.copyItem(at: fileURL, to: destinationURL)
            
            // Upload to server
            let serverItem = try await networkClient.upload(
                fileURL: destinationURL,
                to: parentItemIdentifier,
                progress: progress
            )
            
            // Create item
            let item = FileProviderItem(
                identifier: itemIdentifier,
                parentItemIdentifier: parentItemIdentifier,
                filename: fileURL.lastPathComponent,
                typeIdentifier: UTType(filenameExtension: fileURL.pathExtension)?.identifier ?? UTType.data.identifier,
                capabilities: [.allowsReading, .allowsWriting, .allowsRenaming, .allowsDeleting],
                documentSize: try FileManager.default.attributesOfItem(atPath: fileURL.path)[.size] as? NSNumber,
                contentModificationDate: Date(),
                creationDate: Date(),
                lastUsedDate: nil,
                isMaterialized: true
            )
            
            // Save to database
            database.save(item)
            
            completionHandler(item, nil)
        } catch {
            completionHandler(nil, error)
        }
    }
    
    return progress
}
```

### Modifying Item

```swift
override func modifyItem(
    _ item: NSFileProviderItem,
    baseVersion version: NSFileProviderItemVersion,
    changedFields: NSFileProviderItemFields,
    contents newContents: URL?,
    options: NSFileProviderModifyItemOptions = [],
    request: NSFileProviderRequest,
    completionHandler: @escaping (NSFileProviderItem?, NSFileProviderItemFields, Bool, Error?) -> Void
) -> Progress {
    
    let progress = Progress(totalUnitCount: 100)
    
    Task {
        do {
            var modifiedItem = item
            
            // Handle content changes
            if changedFields.contains(.contents), let newContents = newContents {
                try await networkClient.uploadContents(newContents, for: item.itemIdentifier, progress: progress)
            }
            
            // Handle metadata changes
            if changedFields.contains(.filename) {
                try await networkClient.rename(item.itemIdentifier, to: item.filename)
            }
            
            if changedFields.contains(.parentItemIdentifier) {
                try await networkClient.move(item.itemIdentifier, to: item.parentItemIdentifier)
            }
            
            // Update database
            database.update(modifiedItem)
            
            // Signal enumerator
            signalEnumerator()
            
            completionHandler(modifiedItem, [], false, nil)
        } catch {
            completionHandler(nil, [], false, error)
        }
    }
    
    return progress
}
```

### Creating Directory

```swift
override func createDirectory(
    withName directoryName: String,
    inParentItemIdentifier parentItemIdentifier: NSFileProviderItemIdentifier,
    completionHandler: @escaping (NSFileProviderItem?, Error?) -> Void
) {
    
    Task {
        do {
            // Create on server
            let serverFolder = try await networkClient.createDirectory(name: directoryName, in: parentItemIdentifier)
            
            // Create item
            let folderIdentifier = NSFileProviderItemIdentifier(rawValue: serverFolder.id)
            let item = FileProviderItem(
                identifier: folderIdentifier,
                parentItemIdentifier: parentItemIdentifier,
                filename: directoryName,
                typeIdentifier: UTType.folder.identifier,
                capabilities: [.allowsAddingSubItems, .allowsContentEnumerating, .allowsDeleting, .allowsRenaming],
                documentSize: nil,
                contentModificationDate: Date(),
                creationDate: Date(),
                lastUsedDate: nil,
                isMaterialized: false
            )
            
            // Save to database
            database.save(item)
            
            // Signal enumerator
            signalEnumerator()
            
            completionHandler(item, nil)
        } catch {
            completionHandler(nil, error)
        }
    }
}
```

### Deleting Item

```swift
override func deleteItem(
    identifier: NSFileProviderItemIdentifier,
    baseVersion version: NSFileProviderItemVersion,
    options: NSFileProviderDeleteItemOptions = [],
    request: NSFileProviderRequest,
    completionHandler: @escaping (Error?) -> Void
) -> Progress {
    
    let progress = Progress(totalUnitCount: 100)
    
    Task {
        do {
            // Delete from server
            try await networkClient.delete(identifier)
            
            // Delete local file
            if let localURL = storage.localURL(for: identifier) {
                try? FileManager.default.removeItem(at: localURL)
            }
            
            // Delete from database
            database.delete(identifier)
            
            // Signal enumerator
            signalEnumerator()
            
            completionHandler(nil)
        } catch {
            completionHandler(error)
        }
    }
    
    return progress
}
```

## Error Handling

### Creating Proper Errors

```swift
// No such item
let error = NSError(
    domain: NSFileProviderErrorDomain,
    code: NSFileProviderError.noSuchItem.rawValue,
    userInfo: [
        NSLocalizedDescriptionKey: "The item could not be found",
        NSLocalizedRecoverySuggestionErrorKey: "The item may have been deleted"
    ]
)

// Server unreachable
let error = NSError(
    domain: NSFileProviderErrorDomain,
    code: NSFileProviderError.serverUnreachable.rawValue,
    userInfo: [
        NSLocalizedDescriptionKey: "Cannot reach server",
        NSLocalizedRecoverySuggestionErrorKey: "Check your internet connection and try again"
    ]
)

// Insufficient quota
let error = NSError(
    domain: NSFileProviderErrorDomain,
    code: NSFileProviderError.insufficientQuota.rawValue,
    userInfo: [
        NSLocalizedDescriptionKey: "Not enough storage space",
        NSLocalizedRecoverySuggestionErrorKey: "Free up space or upgrade your storage plan"
    ]
)

// Not authenticated
let error = NSError(
    domain: NSFileProviderErrorDomain,
    code: NSFileProviderError.notAuthenticated.rawValue,
    userInfo: [
        NSLocalizedDescriptionKey: "Authentication required",
        NSLocalizedRecoverySuggestionErrorKey: "Please sign in to continue"
    ]
)
```

## Logging

### Setting up os_log

```swift
import os.log

extension OSLog {
    private static var subsystem = Bundle.main.bundleIdentifier!
    
    static let fileProvider = OSLog(subsystem: subsystem, category: "FileProvider")
    static let enumeration = OSLog(subsystem: subsystem, category: "Enumeration")
    static let network = OSLog(subsystem: subsystem, category: "Network")
    static let database = OSLog(subsystem: subsystem, category: "Database")
}
```

### Using os_log

```swift
// Debug logging
os_log(.debug, log: .fileProvider, "Fetching item: %{public}@", itemIdentifier.rawValue)

// Info logging
os_log(.info, log: .enumeration, "Enumerated %d items", items.count)

// Error logging
os_log(.error, log: .network, "Upload failed: %{public}@", error.localizedDescription)

// Fault logging (critical errors)
os_log(.fault, log: .database, "Database corruption detected")
```

### Viewing Logs in Console.app

1. Open Console.app
2. Select your device/simulator
3. Filter by process: `fileproviderd`
4. Filter by subsystem: `com.yourapp.FileProvider`
5. Use predicates to filter by category or level

## Signaling Enumerator

After making changes, signal the enumerator to refresh:

```swift
func signalEnumerator() {
    let domainIdentifier = NSFileProviderDomainIdentifier(rawValue: "com.myapp.fileprovider")
    guard let manager = NSFileProviderManager(for: domainIdentifier) else { return }
    
    // Signal working set
    manager.signalEnumerator(for: .workingSet) { error in
        if let error = error {
            os_log(.error, log: .fileProvider, "Failed to signal working set: %{public}@", error.localizedDescription)
        }
    }
}
```

Signal specific container:

```swift
manager.signalEnumerator(for: containerIdentifier) { error in
    if let error = error {
        os_log(.error, log: .fileProvider, "Failed to signal container: %{public}@", error.localizedDescription)
    }
}
```
