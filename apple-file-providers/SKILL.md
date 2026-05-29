---
name: apple-file-providers
description: Implement and maintain file provider extensions for apps on Apple operating systems.
license: MIT
metadata: 
    author: i2h3
---

# Apple File Providers

Production-ready code and expert guidance about the file provider framework by Apple for AI.

## General

- Prefer to implement a `NSFileProviderReplicatedExtension` over `NSFileProviderExtension`. The former is the successor to the latter. The latter is only available for iOS. In case of the former, the system takes care of file and folder management, simplifying the implementation and improving performance.
- A file provider extension target must be in the same application group as its shipping main app.

## Concurrency

- Take into consideration that method calls from the file provider framework are on an arbitrary queue and may be concurrent. Ensure thread safety in your code.
- Always prefer to use `await` and `async` functions for asynchronous operations instead of completion handlers.
- Bridge completion handler patterns required by the file provider framework (for example `item(for:request:completionHandler:)` on `NSFileProviderReplicatedExtension`) to a `Task` which then calls `async` and `await` code. The `Progress` objects created and provided to the file provider framework need to be updated consistently with the completion handler call in case of cancellation.

## Data Management

### File Provider Domain Identifiers

- A file provider domain should always use a unique life time identifier based on a `UUID` for itself. Never reuse a previously used `UUID` for a newly added file provider domain. This avoids problems with stale data.
- File provider items should always use a unique identifier based on a `UUID` to identify themselves. Never reuse a previously used `UUID` for a newly created file provider item. This avoids problems with stale data.

### File Provider Item Identifiers

- If file provider items have a unique identifier for the remote item they represent, then that identifier must be stored separately and associated with the local `UUID` of the file provider item. The remote identifier might outlive the local file provider domain identifier or file provider item identifier. This is an additional safety measure to avoid problems with stale data.

### File Provider Item User Data

- Once file provider items have been enumerated by the framework, their `userInfo` dictionary is cached by the framework. Updates to the `userInfo` dictionary can only be achieved by provoking a re-enumeration of the item by the framework by calling `NSFileProviderManager.signalEnumerator(for:completionHandler:)`.

### File Provider Domain Support Data

- If a file provider extension persists data locally, it should use its dedicated sandbox container by default which can be resolved by `URL.library`.
- A file provider extension does not have the permissions to create directories in the group container shared with the main app.
- If a file provider extension persists data locally, it should use a dedicated directory for each file provider domain to store its data. For example: `<container>/Library/Application Support/<file provider domain identifier>/`.

### Data Models

- Data models should always be value types, immutable and conform to `Sendable`.
- Use a dedicated type to implement `NSFileProviderItem` protocol.
- If a file provider item is deleted on the local device, its record must be retained and marked as deleted until the deletion could actually be performed on the remote item.
- If a file provider item is detected as deleted on the remote end, it must also be deleted from the local metadata persistence but only after it has been reported to the file provider framework as deleted. This is required to properly report deletions to the file provider framework and avoid problems with stale data.

## User Interface

- Finder displays the name of the app which manages a file provider domain in the Finder sidebar, assuming there is only one file provider domain by an app. In case there is more than one file provider domain by an app, then Finder displays the app name and the programmatically defined display name of the file provider domain, both separated by a hyphen.
- Updating the display name of a file provider domain requires a NSFileProviderDomain object with the same identifier to be added again by calling `NSFileProviderManager.add(_:completionHandler:)`.
- File provider domain display names must not end with a DNS domain because there are popular top-level domains (for example: `app`) which collide with macOS package and bundle recognition. macOS then misinterprets a file provider domain root directory as a broken app bundle and it cannot be opened like a folder in Finder and elsewhere.
- Custom file provider actions have activation rules which usually query the `userInfo` dictionary of an item cached by the file provider framework. When a new custom file provider action is introduced in a new app release, it will query the cached `userInfo` dictionary of items which were enumerated by the framework in previous app releases. That cached `userInfo` dictionary might not yet contain the expected keys for the new custom file provider action. A re-enumeration of the items is required to update the cached `userInfo` dictionary by calling `NSFileProviderManager.signalEnumerator(for:completionHandler:)`.

## Error Reporting

- When implementing `NSFileProviderEnumerating`, the `func enumerator(for containerItemIdentifier: NSFileProviderItemIdentifier, request: NSFileProviderRequest) throws -> any NSFileProviderEnumerator` method must `throw NSFileProviderError(.noSuchItem)` if the requested `containerItemIdentifier` is not a system defined container identifier like `NSRootContainerItemIdentifier` or does not exist. This is required to properly report errors to the file provider framework and avoid problems with stale data.

## Troubleshooting

- Use the `fileproviderctl` command line tool to troubleshoot and debug your file provider extension independent from the implementation. It allows you to inspect the state of your file provider domains, items, and operations.
- `fileproviderctl evaluate <path to item>` can be used to evaluate the details of a file provider item as enumerated and known to the framework.