# LGTM — Feedback & Bug Reports

**Looks Good To Meow** is an AI code review + CI/CD security platform for
GitHub PRs. This repo is our **public feedback tracker** — the place to
report bugs, request features, or ask questions in the open.

> Product docs live at [docs.looksgoodtomeow.in](https://docs.looksgoodtomeow.in).
> The website is [looksgoodtomeow.in](https://looksgoodtomeow.in).

---

## 📬 Report a bug or request a feature

The fastest way in is straight from the dashboard sidebar. Or open one
of these directly:

- 🐛 **[Report a Bug](https://github.com/tarinagarwal/lgtm-feedback/issues/new?template=bug_report.yml)**
- ✨ **[Suggest a Feature](https://github.com/tarinagarwal/lgtm-feedback/issues/new?template=feature_request.yml)**

Structured forms guide you through the fields we need to actually help
(reproduction steps, versions, surface). Blank issues are disabled —
please pick a template.

**Other channels** (also surfaced in the issue-picker):

| I want to… | Go here |
|---|---|
| Disclose a **security vulnerability** | [Coordinated disclosure policy](https://looksgoodtomeow.in/security) — please don't file public issues |
| Ask about **billing / account / payment** | Email [support@looksgoodtomeow.in](mailto:support@looksgoodtomeow.in) |
| Get **general support** | The [contact form](https://looksgoodtomeow.in/contact) |
| Read **product docs** | [docs.looksgoodtomeow.in](https://docs.looksgoodtomeow.in) |

---

## 🔄 What happens after you file

We run a small automation stack across two repos so nothing falls
through the cracks:

```
      ┌──────────────────────────────┐
      │  tarinagarwal/lgtm-feedback  │  ← this repo (public)
      │                              │
you ─► │  ① You open an issue        │
      │  ② Bot transfers to internal │──┐
      │  ③ Bot confirms + labels     │  │
      └──────────────────────────────┘  │
                                        ▼
                  ┌────────────────────────────────┐
                  │  tarinagarwal/lgtm             │  ← internal (private)
                  │                                │
                  │  ④ Internal issue created,     │
                  │     linked back to your public │
                  │     thread                     │
                  │  ⑤ Team works, closes with     │
                  │     reason + release version   │
                  └────────────────────────────────┘
                                        │
      ┌──────────────────────────────┐  │
      │  tarinagarwal/lgtm-feedback  │◄─┘
      │                              │
      │  ⑥ Bot mirrors closure +     │
      │     posts release version    │
      │  ⑦ Your issue closes with    │
      │     a clear reason           │
      └──────────────────────────────┘
```

### Concretely:

- **Follow-up comments** you add on your public issue **mirror automatically**
  to the internal thread. Reply here — you don't need to know or track the
  internal issue number.
- **Duplicates get cross-linked.** If two people report the same bug,
  we don't create two internal tickets — the second issue gets a
  `duplicate` label + a comment saying it's been linked.
- **Closure updates come back to your issue.** When the internal issue
  closes (fixed / not planned / duplicate), you get a comment on your
  public issue with the reason. If we ship a fix, we tell you which
  version.
- **Reopens roundtrip too.** If we reopen the internal issue for another
  look, your public one reopens with a heads-up comment.

**Public-safety note:** please **don't paste API keys, tokens, customer
data, or private source code** into public issues. If your report needs
sensitive detail, email [support@looksgoodtomeow.in](mailto:support@looksgoodtomeow.in)
instead.

---

## 🏗️ How the automation works (workflow-nerd corner)

Three GitHub Actions workflows run this two-repo dance:

| Workflow | Repo | Trigger | Purpose |
|---|---|---|---|
| `transfer-issues.yml` | this repo | `issues.opened, reopened` | Dedup + create internal issue + stamp a bidirectional marker on the public issue body |
| `mirror-comments.yml` | this repo | `issue_comment.created` | Mirror reporter follow-ups to the internal thread; skip bots, team, and mirror-marker comments to prevent loops |
| `sync-back.yml` | private `lgtm` repo | `issues.closed, reopened, labeled` | Push closure / reopen / release-version updates back to the public issue |

Both tokens are fine-grained PATs scoped to `Issues: R/W` on the
sibling repo only. Neither can read code or bulk-list issues.

**Loop safety:** every mirrored post carries a hidden HTML marker so
the workflows short-circuit on their own posts. Human posts skip the
loop-marker check → get mirrored. Sync-back only fires on issue-level
events (open/close/label), not comments, so mirrored comments don't
retrigger it.

Full workflow source is in [`.github/workflows/`](.github/workflows/).
The sync-back workflow lives in the private repo — its source template
is documented at
[`.github/_internal_ops/SYNC_BACK.md`](.github/_internal_ops/SYNC_BACK.md).

---

## 📦 The product

If you landed here without context: LGTM reviews every GitHub PR with
6 specialist AI agents in parallel (security, bugs, performance,
readability, best-practices, documentation), plus 16 deterministic
CI/CD security detectors that block bad merges at the pipeline gate.

- 🌐 Website: [looksgoodtomeow.in](https://looksgoodtomeow.in)
- 📖 Docs: [docs.looksgoodtomeow.in](https://docs.looksgoodtomeow.in)
- 💻 CLI: `npm i -g @tarin/lgtm-cli`
- 🐈 GitHub App: [github.com/apps/tarin-lgtm](https://github.com/apps/tarin-lgtm/installations/new)

Built by [DevsBazaar](https://looksgoodtomeow.in/about) (Tarin Agarwal,
sole proprietor) from Bangalore, India.
