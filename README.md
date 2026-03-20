# LGTM — Feedback & Bug Reports

**Looks Good To Meow** is an AI-powered code review platform for GitHub pull requests.

This repo is the public feedback tracker. Use it to report bugs, request features, or ask questions.

## What is LGTM?

LGTM reviews every GitHub PR with 6 specialist AI agents running in parallel:

| Agent          | What it checks                                  |
| -------------- | ----------------------------------------------- |
| Security       | Vulnerabilities, secrets, injection risks       |
| Bugs           | Logic errors, null refs, race conditions        |
| Performance    | N+1 queries, memory leaks, slow patterns        |
| Readability    | Naming, complexity, code clarity                |
| Best Practices | Design patterns, error handling, idiomatic code |
| Documentation  | Missing docs, outdated comments, changelog gaps |

A **Synthesizer** then weighs all findings and posts a single verdict (Approved / Changes Requested) directly on your PR as a GitHub comment.

### Key features

- **BYOK (Bring Your Own Key)** — uses your OpenAI or Google Gemini API key. You only pay your provider for tokens used.
- **Tree-sitter + PageRank context** — indexes your codebase structure for context-aware reviews, not just diff-level analysis.
- **CLI for local reviews** — review uncommitted changes before you even push: `npm i -g @tarin/lgtm-cli`
- **Auto-review on PR open** — connect a repo, enable auto-review, and every new PR gets reviewed automatically.
- **PR Chat** — ask follow-up questions about review findings directly in the dashboard.
- **Public review reports** — share a read-only link to any review.

## Links

- **Website**: [looksgoodtomeow.in](https://looksgoodtomeow.in)
- **Docs**: [looksgoodtomeow.in/docs](https://looksgoodtomeow.in/docs)
- **CLI**: [`@tarin/lgtm-cli`](https://www.npmjs.com/package/@tarin/lgtm-cli)
- **GitHub App**: [Install collection-lgtm](https://github.com/apps/tarin-lgtm/installations/new)

## How to report a bug

1. Go to [Issues](../../issues) and click **New Issue**
2. Pick the **Bug Report** template
3. Fill in what happened, what you expected, and steps to reproduce
4. Add screenshots if possible

## How to request a feature

1. Go to [Issues](../../issues) and click **New Issue**
2. Pick the **Feature Request** template
3. Describe the feature and why it would be useful

## Labels

| Label           | Meaning                      |
| --------------- | ---------------------------- |
| `bug`           | Something isn't working      |
| `feature`       | New feature request          |
| `question`      | General question             |
| `cli`           | Related to `@collection/lgtm-cli` |
| `dashboard`     | Related to the web dashboard |
| `review-agents` | Related to AI review quality |

## Contact

- **Email**: tarinagarwal@gmail.com
- **Website**: [looksgoodtomeow.in](https://looksgoodtomeow.in)

---

Built by [Tarin Agarwal](https://tarinagarwal.in)
