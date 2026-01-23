# Apple File Provider Framework Expert Knowledge

Expert guidance about the Apple File Provider framework for iOS and macOS, designed to help AI assistants and developers understand the complexities, pitfalls, and best practices that aren't always obvious from the official documentation.

## Overview

This repository contains comprehensive documentation about Apple's File Provider framework in the form of an [Agent Skills specification](https://agentskills.io/specification). The documentation is based on real-world experience implementing file provider extensions for cloud storage services on iOS and macOS.

## What's Inside

The documentation covers:

- **Fundamentals**: Core concepts, architecture, and lifecycle of file provider extensions
- **Common Pitfalls**: Issues that aren't well-documented and can cause serious problems
- **Platform-Specific Considerations**: Differences between iOS and macOS implementations
- **Best Practices**: Proven patterns for reliable, performant file providers
- **Advanced Topics**: Sync strategies, conflict resolution, and optimization techniques
- **Debugging**: Strategies and tools for debugging file provider extensions
- **Security**: Security and privacy considerations for handling user files

## Structure

The documentation is provided in `agentskills.json` following the Agent Skills specification format. This makes it easy for AI assistants to consume and provide expert guidance on file provider development.

### Key Topics Covered

1. **Extension Lifecycle and Process Management** - Understanding how file provider extensions run as separate processes
2. **Domain Management** - Avoiding common pitfalls with domain registration and management
3. **Enumeration Implementation** - Critical details for implementing NSFileProviderEnumerator correctly
4. **Item Identity and Metadata** - Best practices for NSFileProviderItem implementation
5. **File Content Management** - Fetching, providing, and materializing file contents
6. **Sync Strategies** - Push/pull synchronization patterns
7. **Error Handling** - Using appropriate error codes and providing good user experience
8. **Performance Optimization** - Tips for building fast, efficient file providers
9. **Testing and Debugging** - Comprehensive testing strategies and debugging techniques
10. **Platform Differences** - iOS vs macOS specific considerations

## Usage

### For AI Assistants

Load the `agentskills.json` file to access expert knowledge about the Apple File Provider framework. The structured format allows you to quickly find relevant information about specific topics.

### For Developers

While designed for AI consumption, developers can also read the JSON file directly. Each knowledge entry contains:
- `id`: Unique identifier for the topic
- `title`: Human-readable title
- `category`: Topic category (fundamentals, pitfalls, best-practices, etc.)
- `content`: Detailed explanation with examples and critical information

## Categories

- **fundamentals**: Core concepts and basic implementation
- **pitfalls**: Common mistakes and how to avoid them
- **best-practices**: Recommended patterns and approaches
- **advanced**: Complex topics for experienced developers
- **platform-specific**: iOS or macOS specific considerations
- **debugging**: Tools and techniques for troubleshooting
- **resources**: Additional learning materials

## Why This Documentation?

Apple's official documentation for the File Provider framework is comprehensive but leaves many gaps:
- Extension lifecycle details are vague
- Working set semantics are unclear
- Performance expectations are not specified
- Error recovery patterns are under-documented
- Migration strategies are missing
- Real-world debugging techniques aren't covered

This documentation fills those gaps with practical, experience-based guidance.

## Contributing

This documentation is based on personal experience implementing file provider extensions. If you have additional insights, corrections, or suggestions, contributions are welcome!

## License

MIT License - See LICENSE file for details.

## Disclaimer

This is unofficial documentation based on personal experience. While every effort has been made to ensure accuracy, this is not official Apple documentation. Always refer to Apple's official documentation for the authoritative source.

## Related Resources

- [Apple File Provider Documentation](https://developer.apple.com/documentation/fileprovider)
- [WWDC Sessions on File Provider](https://developer.apple.com/videos/)
- [Agent Skills Specification](https://agentskills.io/specification)
