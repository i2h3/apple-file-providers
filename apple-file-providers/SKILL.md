---
name: apple-file-providers
description: Implement and maintain file provider extensions for apps on Apple operating systems.
license: MIT
metadata: 
    author: i2h3
---

# Apple File Providers

Production-ready code and expert guidance about the file provider framework by Apple for AI.

## Concurrency

- Take into consideration that method calls from the file provider framework are on an arbitrary queue and may be concurrent. Ensure thread safety in your code.
- Always prefer to use `await` and `async` functions for asynchronous operations instead of completion handlers.
- Bridge completion handler patterns required by the file provider framework (in example `item(for:request:completionHandler:)` on `NSFileProviderReplicatedExtension`) to a `Task` which then calls `async` and `await` code.

## Data Management

- If a file provider extension persists data locally, it should use the sandbox container of the file provider extension by default.
- If a file provider extension persists data locally, it should use a dedicated directory for each file provider domain to store its data.
- Data models should always be value types, immutable and conform to `Sendable`.
- Use a dedicated type to implement `NSFileProviderItem` protocol.

## User Interface

- Finder displays the name of the app which manages a file provider domain in the Finder sidebar, assuming there is only one file provider domain by an app. In case there is more than one file provider domain by an app, then Finder displays the app name and the programmatically defined display name of the file provider domain, both separated by a hyphen.
- Updating the display name of a file provider domain requires a NSFileProviderDomain object with the same identifier to be added again through the NSFileProviderManager.

## Troubleshooting

- Use the `fileproviderctl` command line tool to troubleshoot and debug your file provider extension. It allows you to inspect the state of your file provider domains, items, and operations.