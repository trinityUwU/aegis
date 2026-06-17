# Documentation Aegis — index

Corpus de conception et de planification. Ordre de lecture conseillé.

## Vision & produit
- [`product-vision.md`](./product-vision.md) — mission, positionnement, principes, scope in/out, horizon 3 ans.
- [`feature-matrix.md`](./feature-matrix.md) — inventaire exhaustif des capacités × version cible.
- [`roadmap-versions.md`](./roadmap-versions.md) — v0.1 → v3.0, thèmes, contenu, critères de sortie.

## Architecture & specs
- [`../ARCHITECTURE.md`](../ARCHITECTURE.md) — découpage par domaine, frontières, sécurité.
- [`modules.md`](./modules.md) — architecture complète des crates (au-delà du MVP).
- [`ipc-contract.md`](./ipc-contract.md) — événements, verdicts, commandes (`aegis-core`).
- [`detection-catalog.md`](./detection-catalog.md) — menaces traquées par tactique MITRE.
- [`policy-model.md`](./policy-model.md) — modes detection/prevention, réponse graduée, faux positifs.

## Opérations
- [`threat-intel.md`](./threat-intel.md) — feeds, mises à jour signées, IOC, réputation de hash.
- [`ui-spec.md`](./ui-spec.md) — design system, écrans, motion, connexion temps réel.
- [`qa-and-perf.md`](./qa-and-perf.md) — budget perf, bench offensif, corpus, CI, validation E2E.
- [`distribution-and-release.md`](./distribution-and-release.md) — packaging multi-distro, canaux, signature.
- [`security-and-governance.md`](./security-and-governance.md) — threat model produit, anti-tamper, gouvernance OSS.

## Exécution
- Plan d'implémentation : `~/.claude/plans/le-but-de-l-app-abundant-truffle.md` (local, hors repo).
- État courant : [`../STATE.md`](../STATE.md) · Roadmap tâches : [`../TODO.md`](../TODO.md).
