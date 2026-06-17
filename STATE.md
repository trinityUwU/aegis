# STATE — Aegis
*Dernière mise à jour : 2026-06-17*

> Résumé vivant cross-session. Garder < 300 lignes.

## Statut global

**Phase : conception / documentation.** Aucun code applicatif. Le dépôt contient
la vision, l'architecture, la roadmap et les specs fondamentales (IPC, catalogue
de détection, modèle de policy). Prochaine étape exécutable : scaffold du
workspace Cargo et du projet UI (Phase 0, non démarré).

**Conception produit complète** (pas seulement MVP), corpus dans `docs/` (index
`docs/README.md`) :
- Specs techniques : `ipc-contract.md`, `detection-catalog.md`, `policy-model.md`.
- Produit : `product-vision.md` (mission, scope in/out, horizon 3 ans),
  `feature-matrix.md` (inventaire exhaustif × version), `roadmap-versions.md`
  (v0.1 alpha → v1.0 stable → v1.x profondeur → v2.0 chasseur → v3.0 fleet),
  `modules.md` (11 crates cibles).
- Opérations : `threat-intel.md`, `ui-spec.md`, `qa-and-perf.md`,
  `distribution-and-release.md`, `security-and-governance.md`. + `SECURITY.md`.
- Avantage concurrentiel : `competitive-edge.md` — 8 features qui battent les
  concurrents (anti-evasion cross-view, rootkits eBPF, io_uring, drift, deception)
  face au champ de bataille 2026 (VoidLink, RingReaper). Reflété en catégorie Q de
  la feature-matrix.

Toutes les décisions techniques sont déléguées à l'agent — Chris fournit vision et
orchestration, ne tranche pas la technique (délégation actée et durable).

## Décisions figées (2026-06-17)

- **Cible** : poste de travail Linux (desktop, local). Pas serveur/VPS. Perf
  maîtrisée mais on peut se permettre des traitements non triviaux.
- **Nature du produit** : EDR hybride desktop, pas un clone de scanner Windows.
  Signatures (YARA) + comportemental kernel (eBPF + fanotify).
- **Langage** : Rust (daemon + sondes + détection + réponse), TypeScript/React
  (UI uniquement).
- **Interface** : daemon persistant (systemd, privilégié) + UI client de contrôle.
  UI empaquetée en Tauri (pas Electron — surface d'attaque). Le même front est
  servable en webapp via le WebSocket local, gratuitement.
- **API** : locale obligatoire (socket Unix + WebSocket). Externe = non par défaut,
  aucun port réseau ouvert ; architecture qui la permet plus tard, hors MVP.
- **Licence** : MIT. ClamAV (GPLv2) écarté du MVP pour préserver MIT — YARA (BSD)
  suffit. ClamAV éventuel en process séparé post-MVP.
- **IA locale** : écartée pour l'instant (trop lourde, peu pertinente à ce stade).

## Prochaine session — exécution

Plan d'implémentation complet validé : `~/.claude/plans/le-but-de-l-app-abundant-truffle.md`.
Objectif : mise en place concrète. Stack figée — eBPF `aya`, signatures `yara-x`,
fanotify pour blocage fichiers/exec, BPF-LSM (fallback SIGKILL userspace), tokio,
Tauri v2. **Pré-vol obligatoire** : diagnostic kernel (CONFIG_BPF_LSM, lsm=bpf,
BTF, fanotify) + toolchain (nightly, bpf-linker, bun). Si `bpf` absent des LSM
actifs → Chris doit modifier GRUB + reboot. Cible session : Lots 0→1 complets +
Lot 2 (yara-x on-demand) + amorce UI flux live.

Invariants imposés : temps réel exhaustif · budget CPU strict (filtrage in-kernel,
zéro polling) · zéro GPU/VRAM (pas de ML, détection règles+corrélation).

## Garde-fous perf (desktop)

- fanotify ciblé exécution + zones chaudes, pas tout le filesystem.
- eBPF en mode léger (exec + escalade), pas de tracing exhaustif type Tracee.

## Inspirations retenues

- Tetragon — enforcement in-kernel (kill avant exécution du syscall, anti-TOCTOU).
- Falco — moteur de règles comportementales mappées MITRE.
- YARA — signatures fichier + mémoire (fileless).
- Wazuh — FIM déclenchant un scan YARA.
- Linux Malware Detect — modèle multi-moteurs en cascade + quarantaine.

## Différenciateurs visés

1. Fusion signatures + comportemental dans un produit unique et utilisable.
2. UX temps réel soignée sur du runtime-security (inexistant en open source).
3. Souveraineté native, 100 % local, zéro télémétrie.

## Contexte

- Nom : **Aegis** (validé).
- Repo local : `/mnt/projects/aegis`. Repo GitHub public : https://github.com/trinityUwU/aegis (branche `master`).
