# QA & performance — Aegis

Un antivirus qui rate des menaces est inutile ; un antivirus qui ralentit le poste
est désinstallé. Les deux se mesurent, en continu.

## Budget de performance (invariant produit)

Cibles mesurées, pas estimées. Régression = build rejeté.

| Métrique | Cible |
|---|---|
| CPU daemon au repos | < 1 % (idéal ~0,7 %, cf. Tetragon) |
| CPU lors d'un scan on-demand | borné, niçable, interruptible |
| Latence event kernel → userspace | < quelques ms |
| Latence décision sur event fanotify bloquant | très inférieure au timeout kernel (sinon gel) |
| RAM résidente du daemon | budget strict (ordre de quelques dizaines de Mo) |
| GPU / VRAM | **0** — aucune utilisation, jamais |
| Empreinte au boot | démarrage rapide, pas de pic |

Leviers (cf. `plan` d'exécution) : filtrage in-kernel eBPF (ne remonter que le
pertinent), fanotify ciblé (pas de mark récursif sur `/`), yara-x déclenché jamais
en boucle, cache LRU des hash, événementiel pur (zéro polling).

## Bench offensif simulé (inoffensif, reproductible)

Suite de tests dans `tests/redteam/`, exécutée à chaque lot :
- **EICAR** + règle YARA maison → détection signature on-access/on-demand.
- Binaire trivial dans `/tmp` puis exécuté → détection exec zone suspecte.
- Script « ransomware » jouet (réécriture en rafale + canari touché) → kill avant
  propagation.
- Reverse shell de test (`bash -i` redirigé vers socket localhost) → détection C2.
- Programme avec `mmap` anonyme W+X → détection fileless (BPF-LSM ou fallback).
- Faux LOLBin (binaire légitime au comportement anormal) → détection comportementale.
- Écriture cron/systemd/`ld.so.preload` → détection persistance.

## Corpus de détection (lab isolé)
- **Vrais échantillons** (MalwareBazaar) exécutés/scannés **uniquement en VM/sandbox
  isolée jamais connectée**, pour mesurer le **taux de détection**.
- Jamais sur le poste de prod. Procédure de lab documentée séparément.

## Corpus de faux positifs
- Binaires/scripts légitimes courants (gestionnaires de paquets, compilateurs,
  outils dev, langages) → mesurer le **taux de faux positifs**, objectif proche de
  zéro en usage normal. Un faux positif sur un outil dev courant = bug bloquant.

## Tests fonctionnels & qualité de code
- Tests unitaires par crate (logique de détection, corrélation, policy).
- Tests d'intégration du pipeline (event simulé → verdict → réponse).
- `cargo clippy` strict (zéro warning), `cargo fmt`, respect des standards
  (fichier < 500 l., fonction < 35 l., zéro `unwrap` non justifié en chemin chaud).
- Lint standards maison (`/lint`) sur le code TS de l'UI.

## CI (GitHub Actions)
- Build du workspace + UI, clippy, fmt, tests, bench offensif simulé.
- **Bench de régression perf** : échec si le CPU au repos ou la latence dépasse le
  budget. La perf est un test, pas une intention.
- Build des paquets (canaux) sur tag.

## Validation E2E (toujours par le parent, jamais déléguée)
Pour chaque lot livré : lancer le daemon (Log Watcher `process_start`), rejouer le
bench, Playwright sur l'UI (flux live + screenshot lu par Claude),
`process_get_errors` après chaque action. **Zéro erreur console/process** avant de
déclarer un lot terminé. Le motion/visuel reste l'appel de Chris.
