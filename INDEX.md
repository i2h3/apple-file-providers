# Documentation Index

Complete index of all Apple File Provider framework documentation in this repository.

## Quick Links

- [README](README.md) - Overview and introduction
- [Agent Skills JSON](agentskills.json) - Structured documentation for AI consumption
- [Quick Reference](QUICK_REFERENCE.md) - Code examples and snippets
- [Troubleshooting Guide](TROUBLESHOOTING.md) - Solutions to common problems
- [Contributing Guide](CONTRIBUTING.md) - How to contribute to this documentation

## Knowledge Topics (agentskills.json)

### Fundamentals (6 topics)

1. **file-provider-overview** - File Provider Framework Overview
2. **extension-lifecycle** - Extension Lifecycle and Process Management
3. **fetch-contents** - Fetching and Providing File Contents
4. **import-create-modify** - Creating, Importing, and Modifying Items
5. **delete-trash** - Deletion and Trash Handling
6. **sync-push-pull** - Synchronization Patterns

### Common Pitfalls (5 topics)

7. **domain-management** - Domain Management Pitfalls
8. **enumeration-pitfalls** - Enumeration Implementation Pitfalls
9. **item-identity** - Item Identity and Metadata
10. **common-crashes** - Common Crash Scenarios and Fixes
11. **documentation-gaps** - Official Documentation Gaps

### Best Practices (7 topics)

12. **error-handling** - Error Handling Best Practices
13. **testing-strategy** - Testing Strategy and Common Test Cases
14. **performance-optimization** - Performance Optimization Tips
15. **security-considerations** - Security and Privacy Considerations
16. **code-organization** - Code Organization and Architecture
17. **real-world-tips** - Real-World Implementation Tips
18. **thumbnail-metadata** - Thumbnail and Metadata Provider

### Advanced Topics (4 topics)

19. **materialization-eviction** - Materialization and Eviction
20. **migration-strategy** - Version Migration and Updates
21. **background-upload** - Background Upload Implementation
22. **thumbnail-metadata** - Thumbnail and Metadata Provider

### Platform-Specific (2 topics)

23. **ios-specifics** - iOS-Specific Considerations
24. **macos-specifics** - macOS-Specific Considerations

### Debugging (1 topic)

25. **debugging-strategies** - Debugging File Provider Extensions

### Resources (1 topic)

26. **wwdc-sessions** - Essential WWDC Sessions

## Quick Reference Topics (QUICK_REFERENCE.md)

### Basic Setup
- Info.plist Configuration
- Extension Class

### Domain Management
- Adding a Domain (in Main App)
- Removing a Domain
- Getting Manager for Domain

### Enumerator Implementation
- Basic Enumerator
- Working Set Enumerator

### Item Implementation
- NSFileProviderItem Struct
- Creating Items (Files and Folders)

### Content Management
- Fetching Contents
- Providing Item
- Importing Document
- Modifying Item
- Creating Directory
- Deleting Item

### Error Handling
- Creating Proper Errors
- Common Error Codes

### Logging
- Setting up os_log
- Using os_log
- Viewing Logs in Console.app

### Signaling Enumerator
- Signal after changes
- Signal specific containers

## Troubleshooting Topics (TROUBLESHOOTING.md)

### Extension Not Loading
- Extension doesn't appear in Files.app
- Domain registered but not visible in sidebar

### Files Not Appearing
- Files don't show up in Files.app/Finder
- Files appear but show as zero bytes

### Enumeration Issues
- Infinite enumeration loop (100% CPU usage)
- Files appear and disappear randomly

### Upload/Download Problems
- Downloads fail silently
- Uploads don't trigger
- Large file uploads fail or timeout

### Performance Issues
- Slow enumeration
- High memory usage

### Crash Scenarios
- Crash with "Illegal seek"
- Crash in enumeration

### Debugging Tips
- View Extension Logs
- Common Log Predicates
- Attach Debugger
- Inspect Database
- Force Extension Reload
- Test Network Issues
- Extension Logging Best Practices

## Topics by Use Case

### Getting Started
1. [File Provider Overview](agentskills.json#file-provider-overview)
2. [Extension Lifecycle](agentskills.json#extension-lifecycle)
3. [Basic Setup](QUICK_REFERENCE.md#basic-setup)
4. [Domain Management](QUICK_REFERENCE.md#domain-management)

### Implementing Core Features
1. [Item Implementation](QUICK_REFERENCE.md#item-implementation)
2. [Item Identity and Metadata](agentskills.json#item-identity)
3. [Enumerator Implementation](QUICK_REFERENCE.md#enumerator-implementation)
4. [Enumeration Pitfalls](agentskills.json#enumeration-pitfalls)
5. [Content Management](QUICK_REFERENCE.md#content-management)

### File Operations
1. [Creating, Importing, and Modifying Items](agentskills.json#import-create-modify)
2. [Deletion and Trash Handling](agentskills.json#delete-trash)
3. [Fetching and Providing File Contents](agentskills.json#fetch-contents)

### Synchronization
1. [Synchronization Patterns](agentskills.json#sync-push-pull)
2. [Background Upload Implementation](agentskills.json#background-upload)
3. [Materialization and Eviction](agentskills.json#materialization-eviction)

### Error Handling & Debugging
1. [Error Handling Best Practices](agentskills.json#error-handling)
2. [Error Handling Examples](QUICK_REFERENCE.md#error-handling)
3. [Debugging Strategies](agentskills.json#debugging-strategies)
4. [Troubleshooting Guide](TROUBLESHOOTING.md)

### Performance & Optimization
1. [Performance Optimization Tips](agentskills.json#performance-optimization)
2. [Performance Issues](TROUBLESHOOTING.md#performance-issues)
3. [Testing Strategy](agentskills.json#testing-strategy)

### Platform-Specific
1. [iOS-Specific Considerations](agentskills.json#ios-specifics)
2. [macOS-Specific Considerations](agentskills.json#macos-specifics)

### Advanced Topics
1. [Code Organization and Architecture](agentskills.json#code-organization)
2. [Security and Privacy Considerations](agentskills.json#security-considerations)
3. [Migration Strategy](agentskills.json#migration-strategy)
4. [Thumbnail and Metadata Provider](agentskills.json#thumbnail-metadata)

### Common Problems
1. [Common Crash Scenarios](agentskills.json#common-crashes)
2. [Domain Management Pitfalls](agentskills.json#domain-management)
3. [Documentation Gaps](agentskills.json#documentation-gaps)
4. [Extension Not Loading](TROUBLESHOOTING.md#extension-not-loading)
5. [Files Not Appearing](TROUBLESHOOTING.md#files-not-appearing)
6. [Enumeration Issues](TROUBLESHOOTING.md#enumeration-issues)

## Search by Keyword

### A-C
- **App Groups**: iOS-Specifics, Security Considerations
- **Authentication**: Security Considerations, Error Handling
- **Background Processing**: Background Upload, iOS-Specifics
- **Batch Operations**: Performance Optimization, Sync Patterns
- **Caching**: Performance Optimization, Best Practices
- **Conflicts**: Sync Patterns, Real-World Tips
- **Crash**: Common Crashes, Troubleshooting

### D-F
- **Database**: Code Organization, Performance Optimization, Troubleshooting
- **Debugging**: Debugging Strategies, Troubleshooting Guide
- **Domain**: Domain Management, Basic Setup
- **Download**: Fetch Contents, Upload/Download Problems
- **Enumeration**: Enumeration Pitfalls, Enumerator Implementation
- **Error Handling**: Error Handling, Troubleshooting
- **Eviction**: Materialization and Eviction
- **Files.app**: iOS-Specifics, Extension Not Loading
- **Finder**: macOS-Specifics

### I-M
- **Item**: Item Identity, Item Implementation
- **Keychain**: Security Considerations, iOS-Specifics
- **Lifecycle**: Extension Lifecycle
- **Logging**: Logging, Debugging Strategies
- **macOS**: macOS-Specifics, System Extension
- **Materialization**: Materialization and Eviction
- **Memory**: Performance Issues, Performance Optimization
- **Metadata**: Item Identity, Thumbnail and Metadata
- **Migration**: Migration Strategy

### N-S
- **Network**: Upload/Download Problems, Performance Optimization
- **NSFileProviderExtension**: Extension Lifecycle, Basic Setup
- **NSFileProviderItem**: Item Identity, Item Implementation
- **Performance**: Performance Optimization, Performance Issues
- **Progress**: Fetch Contents, Background Upload
- **Security**: Security Considerations
- **Sync**: Synchronization Patterns, Real-World Tips
- **System Extension**: macOS-Specifics

### T-Z
- **Testing**: Testing Strategy, Troubleshooting
- **Thumbnail**: Thumbnail and Metadata, Performance Optimization
- **Trash**: Deletion and Trash Handling
- **Troubleshooting**: Troubleshooting Guide
- **Upload**: Background Upload, Upload/Download Problems
- **URLSession**: Background Upload, Upload/Download Problems
- **Working Set**: Enumeration Pitfalls, Working Set Enumerator

## File Size Reference

```
agentskills.json:    ~26 KB (171 lines, 25 knowledge entries)
QUICK_REFERENCE.md:  ~18 KB (619 lines)
TROUBLESHOOTING.md:  ~19 KB (736 lines)
README.md:           ~5 KB  (90 lines)
CONTRIBUTING.md:     ~7 KB  (278 lines)
INDEX.md:            ~7 KB  (this file)
```

## Updates and Maintenance

This documentation is actively maintained. Last major update: 2026-01

To stay updated:
- Watch the repository for changes
- Check commit history for recent additions
- Contribute your own experiences via Pull Requests

## Related Resources

- [Apple FileProvider Documentation](https://developer.apple.com/documentation/fileprovider)
- [Agent Skills Specification](https://agentskills.io/specification)
- [WWDC Sessions on FileProvider](https://developer.apple.com/videos/)

## How to Use This Documentation

### For AI Assistants
Load `agentskills.json` for structured knowledge that can be easily queried and referenced.

### For Developers
- Start with [README.md](README.md) for an overview
- Use [QUICK_REFERENCE.md](QUICK_REFERENCE.md) for copy-paste examples
- Consult [TROUBLESHOOTING.md](TROUBLESHOOTING.md) when facing issues
- Read [agentskills.json](agentskills.json) for in-depth understanding

### For Contributors
- Read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines
- Use this INDEX.md to find what's already documented
- Add new topics or expand existing ones via Pull Requests
