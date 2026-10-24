# Muse — socle CI self-hosted

## Ce dépôt démarre avec

- **10 workers** self-hosted sur le serveur R630 (Debian 13, 80 cœurs) :
  `r630-muse-01` … `r630-muse-10`, labels `self-hosted, muse-worker, ci-node, Linux, X64`,
  services systemd `actions.runner.BaptisteFaisy-Muse.r630-muse-0N.service`.
- **Gate hermes-lite** (`.github/workflows/hermes-lite.yml` + `.github/hermes/gate.py`)
  tournant sur les workers : check `hermes-lite/verdict`, labels `hermes/approuvee` /
  `hermes/conflit`. En **dry-run** au lancement (variable de dépôt
  `HERMES_LITE_DRY_RUN=1`) : il évalue et étiquette, il ne merge pas.
- **Mergify** (`.mergify.yml`) : file de merge, méthode rebase, batching 1-3,
  labels `urgent` (haute) et `priority-low` (basse). L'App GitHub Mergify doit
  être installée sur le dépôt (reste à faire, geste humain).
- **stale-base-guard** : check informatif de retard de branche.
- **ci-budget** (`.github/workflows/ci-budget.yml` +
  `.github/ci-budget/check_timeouts.py`) : garde-fou du budget CI de
  **5 minutes** (règle user 2026-10-24). Chaque job doit déclarer
  `timeout-minutes: 5` au plus ; un job plus long = CI refusée (le check
  `ci-budget` est exigé par `queue_conditions` dans `.mergify.yml`). Toute
  nouvelle CI suit : `concurrency` + `cancel-in-progress`, `fetch-depth: 1`,
  filtres `paths:`, cache. Build natif / suite longue : hors PR (tag,
  `main`, ou `workflow_dispatch`), timeout justifié en commentaire.
  Détail : `~/Bureau/CI-BUDGET-5MIN.md`.

## Mise en service (ordre)

1. (fait par l'agent) workers enregistrés et en ligne ;
2. (fait le 24/10/2026) app GitHub **Mergify** installée sur le compte, accès
   « All repositories » — le produit Merge Queue s'active alors seul ;
3. retirer la variable `HERMES_LITE_DRY_RUN` si l'on veut que le gate hermes-lite
   merge lui-même (sinon Mergify reste le seul pilote de merge — contrat duello
   `MERGIFY_QUEUE_ENABLED` 2026-10-10 : **un seul pilote de merge**).
