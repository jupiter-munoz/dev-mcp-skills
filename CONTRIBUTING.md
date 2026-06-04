# Contributing to Appian MCP Skills

We welcome contributions from anyone building with Appian and AI coding assistants. Whether you're an Appian employee, a partner, or a practitioner — if you've found a pattern that makes the AI more reliable, we want it here.

## How to Contribute

1. **Fork** this repository
2. **Create a branch** in your fork (`git checkout -b my-skill-improvement`)
3. **Make your changes** (see guidelines below)
4. **Commit with DCO sign-off** (see below)
5. **Open a Pull Request** against `main`

## What Makes a Good Contribution

### Evidence of impact

Every skill change should address a real failure mode. In your PR description, include:

- **What went wrong** — describe the error, retry loop, or incorrect output the AI produced
- **Why** — what knowledge was the AI missing?
- **How this fixes it** — what does the skill now teach that prevents the failure?

You don't need to provide formal metrics, but "I hit this problem, added this content, and the AI stopped making the mistake" is the bar.

### Scope

- **One concern per PR** — don't bundle unrelated fixes
- **Fold small additions into existing skills** — if your contribution is <30 lines and closely related to an existing skill's topic, add it there rather than creating a new file
- **Create a new skill** when the content is >50 lines or covers a distinct concern not addressed by existing skills

### Quality

- Write for an AI audience — be precise and explicit, not conversational
- Include correct code examples where applicable
- State what NOT to do when there's a common wrong approach
- Use the existing skills as style examples

## Skill File Format

```
skills/appian-<topic>/
  SKILL.md
  references/       (optional)
    *.md
```

`SKILL.md` must have YAML frontmatter:

```yaml
---
name: "appian-<topic>"
description: "One sentence describing when the AI should load this skill."
---
```

The description is critical — it's how the AI decides whether to load the skill for a given task. Make it specific about the use case, not generic.

## DCO Sign-Off

This project uses the [Developer Certificate of Origin](https://developercertificate.org/) (DCO). You must sign off every commit to certify you have the right to submit it:

```bash
git commit -s -m "Add grid filtering quirk to appian-sail"
```

This adds a `Signed-off-by: Your Name <your@email.com>` line. If you forget, amend:

```bash
git commit --amend -s --no-edit
```

All commits in a PR must have sign-off or the PR cannot be merged.

## Review Process

1. A maintainer will review your PR, usually within a few business days
2. We may ask for evidence of the failure mode or suggest folding content into a different skill
3. Once approved, we'll squash-merge to keep `main` history clean
4. Your contribution will be attributed in the squash commit message

## Reporting Issues

If you find a skill that teaches something **incorrect** (causes failures rather than preventing them), open an issue with:

- Which skill and which section
- What the skill says vs. what actually works
- Your Appian version (if relevant)

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). Be respectful and constructive.
