# Contributing to Apple File Provider Documentation

Thank you for your interest in contributing to this documentation! This guide will help you add your own experiences and insights about the Apple File Provider framework.

## How to Contribute

### Adding New Knowledge Entries

The main documentation is in `agentskills.json`. Each entry follows this structure:

```json
{
  "id": "unique-identifier",
  "title": "Human-Readable Title",
  "category": "category-name",
  "content": "Detailed content with examples and explanations"
}
```

### Categories

Use one of these existing categories:
- **fundamentals**: Basic concepts and core implementation
- **pitfalls**: Common mistakes and how to avoid them
- **best-practices**: Recommended patterns and approaches
- **advanced**: Complex topics for experienced developers
- **platform-specific**: iOS or macOS specific considerations
- **debugging**: Tools and techniques for troubleshooting
- **resources**: Additional learning materials

### Writing Guidelines

1. **Be Specific**: Provide concrete examples and code snippets
2. **Be Practical**: Focus on real-world scenarios and solutions
3. **Be Clear**: Explain why something is a problem, not just what to do
4. **Be Accurate**: Test your suggestions before documenting them
5. **Mark Critical Info**: Use "CRITICAL:", "Best Practice:", "Example:" markers

### Content Structure

Good entries typically include:
- Clear problem or concept statement
- Why it matters
- How to implement/solve it
- Code examples when applicable
- Common mistakes to avoid
- References to official docs (if applicable)

### Example Entry

```json
{
  "id": "batch-operations",
  "title": "Batch Operations for Efficiency",
  "category": "best-practices",
  "content": "When syncing multiple files, batch operations improve performance:\n\n1. **Group API Calls**: Instead of individual requests, batch them\n2. **Batch Size**: Optimal batch size is 50-100 items\n3. **Parallel Execution**: Process batches in parallel with concurrency limit\n\nExample:\n```swift\nfunc syncItems(_ items: [FileProviderItem]) async throws {\n    let batches = items.chunked(into: 50)\n    \n    for batch in batches {\n        try await withThrowingTaskGroup(of: Void.self) { group in\n            for item in batch {\n                group.addTask {\n                    try await self.sync(item)\n                }\n            }\n            try await group.waitForAll()\n        }\n    }\n}\n```\n\nBest Practice: Monitor API rate limits and adjust batch size accordingly."
}
```

## Adding Code Examples

For `QUICK_REFERENCE.md`:
- Add working, tested Swift code
- Include comments for clarity
- Show both what to do and what NOT to do
- Include imports and context

## Adding Troubleshooting Guides

For `TROUBLESHOOTING.md`:
- Start with the symptom/problem
- List possible causes
- Provide step-by-step solutions
- Include diagnostic commands
- Show how to verify the fix

## Testing Your Contributions

Before submitting:
1. Validate JSON syntax: `python3 -m json.tool agentskills.json`
2. Check for typos and grammar
3. Verify code examples compile
4. Test solutions on actual projects if possible

## Pull Request Process

1. Fork the repository
2. Create a feature branch: `git checkout -b add-new-knowledge`
3. Make your changes
4. Validate JSON: `python3 -m json.tool agentskills.json > /dev/null`
5. Commit with clear message: `git commit -m "Add guidance on [topic]"`
6. Push to your fork: `git push origin add-new-knowledge`
7. Create a Pull Request with description of what you added

## Pull Request Template

```markdown
## What does this PR add?

Brief description of the new knowledge/fix

## Category

- [ ] fundamentals
- [ ] pitfalls
- [ ] best-practices
- [ ] advanced
- [ ] platform-specific
- [ ] debugging
- [ ] resources

## Checklist

- [ ] JSON is valid
- [ ] Code examples are tested
- [ ] Content is clear and specific
- [ ] Follows writing guidelines
- [ ] Added entry to appropriate documentation file
```

## Style Guide

### Writing Style

- Use second person ("you") for instructions
- Use present tense
- Be concise but complete
- Use active voice
- Use numbered lists for sequential steps
- Use bullet points for non-sequential items

### Code Style

- Follow Swift API Design Guidelines
- Use meaningful variable names
- Include error handling
- Add comments for complex logic
- Show imports when necessary

### Formatting

- Use proper markdown formatting
- Code blocks should specify language: ```swift
- Use **bold** for emphasis
- Use `code` for identifiers, filenames, and commands
- Use > for important notes

## Common Pitfalls to Avoid

1. **Don't**: Copy-paste from official docs verbatim
   **Do**: Add your experience and interpretation

2. **Don't**: Add untested code examples
   **Do**: Test code before including it

3. **Don't**: Use vague descriptions like "might cause issues"
   **Do**: Be specific about what issues and when

4. **Don't**: Assume knowledge level
   **Do**: Explain concepts clearly for various skill levels

5. **Don't**: Ignore error handling in examples
   **Do**: Show proper error handling patterns

## Topics Needed

We're especially looking for contributions in these areas:

- [ ] CloudKit integration patterns
- [ ] Conflict resolution strategies for specific scenarios
- [ ] Performance profiling techniques
- [ ] Unit testing strategies for file providers
- [ ] SwiftUI integration patterns
- [ ] Combine/async-await patterns
- [ ] App Store review considerations
- [ ] Accessibility implementation
- [ ] Internationalization/localization
- [ ] App Group security best practices
- [ ] Background processing limits
- [ ] Handling large directories (10k+ files)
- [ ] Symbolic link handling
- [ ] Sparse file support
- [ ] File system attributes preservation
- [ ] Extended attributes handling
- [ ] Migration from older file provider APIs

## Questions?

If you're unsure about anything:
1. Open an issue with your question
2. Look at existing entries for examples
3. Start with small contributions
4. Ask for feedback early

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Recognition

All contributors will be acknowledged in the project. Thank you for helping make this documentation better!

## Review Process

- All PRs will be reviewed for accuracy and clarity
- Reviewers may request changes or clarifications
- Once approved, your contribution will be merged
- You'll be added to the contributors list

## Code of Conduct

- Be respectful and constructive
- Focus on improving the documentation
- Welcome newcomers and answer questions kindly
- Credit sources and inspirations

Thank you for contributing to the Apple File Provider community! 🎉
