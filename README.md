# opsagent-prod-firestore-rules

Canonical Firestore ruleset for the **shared** `opsagent-prod` Firestore database.

## Why this repo exists
`opsagent-prod` has ONE Firestore database and ONE ruleset, shared by every app
(lead-pipeline, claude-architect-training, future apps). Whichever repo runs
`firebase deploy --only firestore:rules` last **replaces the entire ruleset**.
On 2026-05-28 a trainer deploy shipped trainer-only rules → an hour later a
lead-pipeline deploy silently overwrote them → trainer signup outage.

This repo is the single source of truth. Every app fetches `firestore.rules`
from here at deploy time instead of carrying its own diverging copy.

**Firestore rules are not secret** (they are enforced server-side but their
content is client-inferrable) — hence this repo is public so WIF-authed CI
deploys can `curl` it without a GitHub token.

## Convention
- `firestore.rules` MUST contain the **union** of every app's collection rules.
- Adding a new app under opsagent-prod? Append its rule block here (clearly
  commented with the owning app) BEFORE the app's first deploy.
- Never delete another app's block.

## Consuming apps (fetch-at-deploy)
Each app's deploy workflow adds, before `firebase deploy`:

```bash
curl -fsSL https://raw.githubusercontent.com/OpsAgentsAI/opsagent-prod-firestore-rules/main/firestore.rules -o firestore.rules
```

The committed `firestore.rules` in each app repo stays as a fallback only.

### Wired apps
- [x] MSApps-Mobile/claude-architect-training (PR pending)
- [ ] OpsAgentsAI/msapps-lead-pipeline (follow-up)

## Owner
/opsagents-cto (architecture) → /gcloud-devops-expert (deploy wiring).
Decision: Option α (shared config repo), card 6a1818a4 on board t22SF8q5.
