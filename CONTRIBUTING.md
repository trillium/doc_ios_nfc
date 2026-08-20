# Contributing to iOS NFC Documentation

Thank you for helping improve this reference library for agents and developers!

## How to Contribute

### Found a Bug or Gap?

1. **Check existing issues first** — your question might already be answered
2. **Open an issue** describing:
   - What you were trying to do
   - What documentation you looked at
   - What was missing or unclear
   - iOS version, device model, tag type if relevant

### Found a Solution?

1. **Document your discovery** in an issue or discussion
2. **Submit a PR** with:
   - Proposed change (code, example, or documentation update)
   - Why this matters (use case, common pattern, etc.)
   - Testing notes if applicable

### Adding a New Pattern

If you discover a pattern not yet covered:

1. **Create an issue** with label `enhancement`
   - Title: "Add pattern: [description]"
   - Body: Explain the use case and why it's important
2. **Submit PR** with:
   - Code example (tested, functional)
   - Where to add it (which doc file)
   - Explanation of when agents should use this

### Improving Existing Docs

- **Clarity issues:** Flag unclear sections in an issue
- **Typos/errors:** Submit PR with fix
- **Missing examples:** Open issue with specific request
- **Outdated iOS info:** Verify against Apple docs, then update

## PR Process

1. **Fork and branch:** Create a feature branch from main
2. **Make changes:** Update docs, add examples, clarify language
3. **Test content:** If code examples, verify they compile/run
4. **Commit message:** Clear, specific commit message
   - ✓ "Add: Universal Links offline behavior testing guide"
   - ✗ "update docs"
5. **Open PR:** Include:
   - What changed and why
   - Any related issues
   - Testing steps if relevant

## Documentation Standards

### Code Examples

- Must be compilable Swift 5.0+
- Include realistic error handling
- Add comments explaining key parts
- Keep examples focused on one pattern

### Tables and Specifications

- Use markdown tables for compatibility matrices
- Include iOS version ranges
- Note device requirements (iPhone 7+, etc.)
- Mark missing/unreliable features with ⚠️

### Warnings and Important Notes

Use these markers:
- `⚠️` — Critical limitation or gotcha
- `✓` — Works/supported
- `✗` — Doesn't work/unsupported
- `→` — Next step/related section

### Cross-References

Link to related sections:
- `[Getting Started](./docs/GETTING_STARTED.md)`
- `[Code Examples](./docs/CODE_EXAMPLES.md)`
- `[Technical Reference](./docs/TECHNICAL.md)`

## Review Process

- All contributions welcome
- Issues/PRs reviewed within 2-3 days
- Maintainers may request:
  - Clarity improvements
  - Additional examples
  - Testing verification
  - iOS version confirmation

## Issues That Need Help

Look for issues labeled:
- `good-first-issue` — Perfect starting point
- `help-wanted` — Specific guidance needed
- `enhancement` — Feature/pattern requests

## Questions?

- 💬 Ask in a GitHub discussion
- 🐛 File an issue with your question
- 📖 Check existing docs first

---

**By contributing, you're helping agents implement NFC features more reliably. Thank you!**
