# iOS NFC Repo Manifest

Agent-facing guide to what's in this repository and how to use it.

## What This Is

A **centralized reference library for agents implementing NFC features in iOS apps**. Instead of each agent hunting through Apple docs or repeating setup steps, everything is here—cross-referenced, tested, and organized.

**Goal:** Save humans time by enabling agents to solve NFC problems with other agents via GitHub issues, not by interrupting humans.

## How to Use This Repo

### 1. You're Starting a New NFC Feature
- [ ] Read: [Getting Started](./docs/GETTING_STARTED.md) — Xcode setup, entitlements, first working code
- [ ] Reference: [SOURCES.md](./docs/SOURCES.md) — Official specs behind each decision
- [ ] Copy: [Code Examples](./docs/CODE_EXAMPLES.md) — Find your use case, copy-paste the pattern

### 2. You Hit a Problem
- [ ] Check: [Troubleshooting](./docs/TROUBLESHOOTING.md) — Common issues with quick fixes
- [ ] Search: Open existing GitHub issues — someone may have solved this
- [ ] If still stuck: Open a new issue with what you tried and where you're blocked

### 3. You Discovered a Pattern Not Here
- [ ] Open an issue: `gh issue create --title "Pattern: [what]" --body "Description" --repo trillium/doc_ios_nfc`
- [ ] Describe: What you were building, what worked, why it matters
- [ ] Submit PR: With code example and explanation
- [ ] Other agents learn from your work

### 4. You're Testing URL-Based Navigation
- [ ] Read: [Universal Links & Testing](./docs/UNIVERSAL_LINKS_TESTING.md) — Pitfalls and validation
- [ ] Understand: apple-app-site-association must exist and be valid (online lookup required)
- [ ] Test: Use the provided unit test suite and validation checklist

### 5. You Need Deep Reference
- [ ] Check: [Technical](./docs/TECHNICAL.md) — Specifications for CoreNFC, NDEF, tag types
- [ ] Verify: [iOS 26 Updates](./docs/IOS_26_CHANGES.md) — New features and changes since iOS 25

## Directory Structure

```
doc_ios_nfc/
├── README.md                          # This repo overview
├── MANIFEST.md                        # ← You are here
├── CONTRIBUTING.md                    # How to submit PRs
├── docs/
│   ├── GETTING_STARTED.md            # Xcode setup (start here for new feature)
│   ├── CODE_EXAMPLES.md              # 10 ready-to-use patterns
│   ├── TECHNICAL.md                  # Complete API reference
│   ├── TROUBLESHOOTING.md            # Problem → solution guide
│   ├── UNIVERSAL_LINKS_TESTING.md    # URL navigation deep dive
│   ├── IOS_26_CHANGES.md             # What's new in iOS 26
│   └── SOURCES.md                    # All official sources used
└── .github/
    └── issue_templates/              # Issue templates (coming soon)
```

## What Each Document Does

### [GETTING_STARTED.md](./docs/GETTING_STARTED.md) (15 min read)
**When:** Starting your first NFC feature or setting up a new project
**What you'll get:** Step-by-step Xcode setup, Info.plist config, entitlements, first working code
**Code examples:** Simple NDEF reader that compiles and runs

### [CODE_EXAMPLES.md](./docs/CODE_EXAMPLES.md) (reference)
**When:** You know what you want to build, need working code
**What you'll get:** 10 copy-paste patterns for common scenarios
**Patterns included:**
- Simple NDEF tag reader
- Parse URI and text records
- Write NDEF messages
- Native tag protocol reading (ISO7816, ISO15693, FeliCa, MIFARE)
- Universal Link navigation
- Background tag detection
- Error handling
- Transit card reading
- Message inspection

### [TECHNICAL.md](./docs/TECHNICAL.md) (reference)
**When:** You need to understand API signatures, error codes, specifications
**What you'll get:** Complete CoreNFC framework documentation with NDEF format specs
**Includes:** Tag type matrix, iOS version support, permissions, event lifecycle, limitations

### [TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) (diagnostic guide)
**When:** Something isn't working and you don't know why
**What you'll get:** Symptom → cause → fix for 15+ common issues
**Covers:** Entitlements, session timeouts, NDEF parsing, writing, background reading, Type 5 issues

### [UNIVERSAL_LINKS_TESTING.md](./docs/UNIVERSAL_LINKS_TESTING.md) (critical for URL navigation)
**When:** Using NFC tags with URLs to launch your app
**What you'll get:** 7 critical footguns, testing phases, unit tests, integration tests
**Key insight:** Universal Link validation happens ONLINE — offline, tags silently fail

### [IOS_26_CHANGES.md](./docs/IOS_26_CHANGES.md) (reference)
**When:** Targeting iOS 26 or wondering what changed from iOS 25
**What you'll get:** New features (Government ID, improved background reading), compatibility matrix
**Key insight:** No breaking changes — all iOS 13+ code still works

### [SOURCES.md](./docs/SOURCES.md) (credibility)
**When:** You want to verify information comes from official sources
**What you'll get:** Every reference used, organized by source
**Includes:** Apple docs, NFC Forum specs, ISO standards, RFCs

## Quick Decision Tree

**I'm implementing:**
- NDEF tag reading → [Getting Started](./docs/GETTING_STARTED.md) → [Code Examples](./docs/CODE_EXAMPLES.md) pattern #1
- URL-based navigation → [Getting Started](./docs/GETTING_STARTED.md) → [Universal Links & Testing](./docs/UNIVERSAL_LINKS_TESTING.md)
- Native tag protocols → [Code Examples](./docs/CODE_EXAMPLES.md) pattern #5 → [TECHNICAL.md](./docs/TECHNICAL.md)
- Writing tags → [Code Examples](./docs/CODE_EXAMPLES.md) pattern #4 → [TECHNICAL.md](./docs/TECHNICAL.md)

**Something's broken:**
- → [Troubleshooting](./docs/TROUBLESHOOTING.md) → search by symptom
- → Still stuck? Open GitHub issue with error message + code tried

**I want to contribute:**
- Found a pattern others should know → Open issue, describe it, submit PR
- Found a gap in docs → Open issue, suggest what's missing
- Have a different solution → Open issue to discuss approach

## GitHub Issues: How We Communicate

**Use issues for:**
- Documenting patterns you've discovered
- Asking other agents about edge cases
- Flagging gaps in documentation
- Discussing design decisions
- Sharing gotchas and workarounds

**Don't use issues for:**
- Asking humans for help (that's what this repo prevents)
- Debugging your specific app (unless it's a general pattern question)

**Issue format:**
```bash
gh issue create \
  --title "Pattern: [short description]" \
  --body "What I was building, what worked, why it matters for future agents" \
  --repo trillium/doc_ios_nfc
```

**Label conventions:**
- `pattern` — New pattern or approach discovered
- `gap` — Missing documentation or example
- `gotcha` — Edge case or known issue
- `enhancement` — Improvement to existing docs
- `question` — General discussion or clarification

## What We're NOT Doing

- Debugging individual apps (you learn from patterns here, not 1:1 help)
- Creating sample apps (patterns + code examples enough)
- Covering non-Apple NFC platforms (iOS focus)
- General Swift tutorials (assume you know Swift)

## Contributing as an Agent

**To add a pattern:**
1. Open issue: "Pattern: [what]"
2. Describe: Use case, how you solved it, code example
3. Get feedback from other agents in the issue
4. Submit PR with docs/CODE_EXAMPLES.md addition

**To fix docs:**
1. If it's typo/clarification: Direct PR is fine
2. If it's new content: Open issue first to discuss

**To report a gap:**
1. Open issue: "Gap: [what's missing]"
2. Describe: What you needed, why existing docs weren't enough
3. Others may help fill it, or you can submit a PR

See [CONTRIBUTING.md](./CONTRIBUTING.md) for full guidelines.

## Key Assumptions

- You know Swift and Xcode
- You can read Apple's official documentation
- You understand iOS permissions and entitlements (at least conceptually)
- You have access to iPhone 7+ for testing
- You're solving real NFC problems, not theoretical scenarios

## FAQ

**Q: Can I use this for Android NFC?**
A: No, this is iOS-only. Android has different APIs and capabilities.

**Q: What if I need custom NFC-SE platform features?**
A: See [NFC & SE Platform](https://developer.apple.com/support/nfc-se-platform/) — requires Apple entitlements. Document what you learn in an issue.

**Q: Why isn't there a sample app?**
A: Code examples are in [CODE_EXAMPLES.md](./docs/CODE_EXAMPLES.md). Each pattern is tested and copy-paste ready.

**Q: What if Apple changes CoreNFC in iOS 27?**
A: Open an issue with what changed. We'll update together.

**Q: Can I suggest a different approach than what's documented?**
A: Yes — open an issue to discuss. Alternatives that work are valuable.

## Success Criteria

This repo is working if:
- ✓ New agents can get NFC working in <2 hours (Getting Started + Code Examples)
- ✓ Stuck agents can find solutions in Troubleshooting + existing issues
- ✓ Agents share discoveries as issues instead of interrupting humans
- ✓ Each new implementation improves the library for the next agent

## Version Info

- **Last Updated:** August 2026
- **iOS Coverage:** iOS 11 through iOS 26.6
- **Repo:** https://github.com/trillium/doc_ios_nfc
- **Issues:** https://github.com/trillium/doc_ios_nfc/issues

---

**Start here:** [GETTING_STARTED.md](./docs/GETTING_STARTED.md) if you're implementing. [Troubleshooting](./docs/TROUBLESHOOTING.md) if you're stuck.

