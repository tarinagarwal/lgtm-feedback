# Sync-back workflow — install in the PRIVATE lgtm repo

The `sync-back.yml` file below lives in the **private** `tarinagarwal/lgtm`
repo (NOT in this feedback repo). It closes the loop: when the team
closes / reopens / labels an internal issue that originated from a
public feedback issue, the change is mirrored back to the public thread
so the reporter sees what happened.

## Install steps

1. In the private `tarinagarwal/lgtm` repo, create the file
   `.github/workflows/sync-back.yml` with the content shown below.

2. On that repo, add a **fine-grained personal access token** with:
   - Repository access: `tarinagarwal/lgtm-feedback` only
   - Repository permissions: `Issues: Read and write`
   - Save it as an Actions secret named `FEEDBACK_SYNC_TOKEN`
   (Distinct from the `TRANSFER_TOKEN` used by the public repo — this
   one points the other direction.)

3. Commit and push. Test by closing an internal issue that was mirrored
   from a public one — the public issue should get a comment + close
   within ~15 seconds.

## The workflow itself

Paste this verbatim into the private repo's `.github/workflows/sync-back.yml`:

```yaml
name: Mirror internal state back to public feedback

# When the team acts on an internal issue that was mirrored from the
# public feedback repo, propagate the state change back to the public
# issue so the reporter isn't left in the dark.
#
# We look up the public issue number by regex-parsing the provenance
# line that the transfer workflow adds:
#     > **Copied from** tarinagarwal/lgtm-feedback#42
#
# Triggers:
#   - closed:   comment + close public issue with the state_reason
#   - reopened: comment + reopen public issue
#   - labeled:  if labeled with a release-marker (e.g. "released:v3.1.0"),
#               reflect that on the public issue

on:
  issues:
    types: [closed, reopened, labeled]

concurrency:
  group: sync-back-${{ github.event.issue.number }}
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  mirror-state:
    runs-on: ubuntu-latest
    steps:
      - name: Sync internal state back to public feedback
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.FEEDBACK_SYNC_TOKEN }}
          script: |
            const PUBLIC_OWNER = "tarinagarwal";
            const PUBLIC_REPO = "lgtm-feedback";
            const internalIssue = context.payload.issue;
            const eventAction = context.payload.action;

            // ── Extract public issue number from provenance line ──
            const provRe = new RegExp(
              `> \\*\\*Copied from\\*\\* ${PUBLIC_OWNER}/${PUBLIC_REPO}#(\\d+)`,
            );
            const match = (internalIssue.body || "").match(provRe);
            if (!match) {
              core.info(`Internal #${internalIssue.number} has no public origin — skipping.`);
              return;
            }
            const publicNumber = Number(match[1]);
            core.info(`Internal #${internalIssue.number} ↔ public #${publicNumber} (${eventAction})`);

            // ── Fetch public issue to ensure it still exists ──
            let publicIssue;
            try {
              ({ data: publicIssue } = await github.rest.issues.get({
                owner: PUBLIC_OWNER,
                repo: PUBLIC_REPO,
                issue_number: publicNumber,
              }));
            } catch (err) {
              core.warning(`Public issue not found: ${err.message}`);
              return;
            }

            // ── Handle each event type ──
            if (eventAction === "closed") {
              const reason = internalIssue.state_reason || "completed";
              const closureBlurb =
                reason === "not_planned"
                  ? "We looked at this and decided not to build it. Details on the internal thread — happy to discuss further here if you disagree."
                  : reason === "duplicate"
                    ? "Marked as duplicate of another internal report. This will move with the tracking issue."
                    : "Fixed / shipped. Thanks for reporting.";

              // Version label search — if the team tagged the internal
              // issue with something like "released:v3.1.0", surface it.
              const versionLabel = (internalIssue.labels || [])
                .map(l => (typeof l === "string" ? l : l.name))
                .find(name => /^released[:/@]v?\d/i.test(name));
              const versionLine = versionLabel
                ? `\\n\\n**Released in:** \`${versionLabel.replace(/^released[:/@]/i, "")}\``
                : "";

              const commentBody = [
                `Update from the team: **closed** as \`${reason}\`.`,
                ``,
                closureBlurb + versionLine,
                ``,
                `_(Auto-mirrored from the internal tracker.)_`,
              ].join("\\n");
              await github.rest.issues.createComment({
                owner: PUBLIC_OWNER,
                repo: PUBLIC_REPO,
                issue_number: publicNumber,
                body: commentBody,
              });
              if (publicIssue.state === "open") {
                await github.rest.issues.update({
                  owner: PUBLIC_OWNER,
                  repo: PUBLIC_REPO,
                  issue_number: publicNumber,
                  state: "closed",
                  state_reason: reason,
                });
              }
              core.info(`Closed public #${publicNumber} with reason=${reason}`);
              return;
            }

            if (eventAction === "reopened") {
              await github.rest.issues.createComment({
                owner: PUBLIC_OWNER,
                repo: PUBLIC_REPO,
                issue_number: publicNumber,
                body: "Update from the team: reopened for another look. We'll comment again when there's news.",
              });
              if (publicIssue.state === "closed") {
                await github.rest.issues.update({
                  owner: PUBLIC_OWNER,
                  repo: PUBLIC_REPO,
                  issue_number: publicNumber,
                  state: "open",
                });
              }
              core.info(`Reopened public #${publicNumber}`);
              return;
            }

            if (eventAction === "labeled") {
              const addedLabel = context.payload.label?.name || "";
              // Only surface release-tag labels; noisy to mirror every
              // internal triage label.
              if (!/^released[:/@]v?\d/i.test(addedLabel)) {
                core.info(`Non-release label "${addedLabel}" — skipping.`);
                return;
              }
              const version = addedLabel.replace(/^released[:/@]/i, "");
              await github.rest.issues.createComment({
                owner: PUBLIC_OWNER,
                repo: PUBLIC_REPO,
                issue_number: publicNumber,
                body: `Update from the team: **released in \`${version}\`**. If your original problem persists, please reopen with steps and we'll take another look.`,
              });
              core.info(`Announced release ${version} on public #${publicNumber}`);
              return;
            }
```

## Testing after install

```bash
# 1) Open a test issue on the public feedback repo
gh issue create -R tarinagarwal/lgtm-feedback \\
  --title "TEST: sync-back roundtrip" \\
  --body "Testing bidirectional workflow. Safe to close."

# 2) Verify transfer created a matching internal issue
gh issue list -R tarinagarwal/lgtm --search "TEST: sync-back"

# 3) Close the internal issue with a fixed-completed reason
gh issue close <internal-issue-number> -R tarinagarwal/lgtm --reason completed

# 4) Verify public issue got closed + a "fixed" comment within ~15s
gh issue view <public-issue-number> -R tarinagarwal/lgtm-feedback --comments

# 5) Cleanup
# Delete both issues from their respective repos when done testing.
```

## Loop-safety

- The transfer workflow (public → private) does NOT fire on `on: issues.closed`,
  so this workflow closing the public issue does not cascade.
- This workflow does NOT fire on `on: issue_comment`, so the mirror-comments
  workflow's mirrored posts to the internal repo don't retrigger sync-back.
- Both workflows short-circuit if the origin/destination marker is missing.
