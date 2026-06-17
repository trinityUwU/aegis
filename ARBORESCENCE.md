# Arborescence — Aegis

Une ligne par fichier. Le code applicatif n'existe pas encore : la section
**Existant** liste le présent, **Structure prévue** documente la cible Phase 0+.

## Existant

```
aegis/
├── LICENSE              # licence MIT
├── README.md            # présentation, stack, principes
├── SECURITY.md          # politique de divulgation des vulnérabilités
├── ARCHITECTURE.md      # domaines, frontières de modules, sécurité
├── STATE.md             # état vivant cross-session
├── TODO.md              # roadmap MVP + backlog
├── ARBORESCENCE.md      # ce fichier
├── .echoforge.yml       # métadonnées projet EchoForge
├── .gitignore           # exclusions git
├── .env.example         # variables d'environnement (vide de secrets)
├── docs/
│   ├── README.md               # index de la documentation
│   ├── product-vision.md       # mission, positionnement, scope, horizon 3 ans
│   ├── feature-matrix.md       # inventaire exhaustif des capacités × version
│   ├── roadmap-versions.md     # v0.1 → v3.0, thèmes, critères de sortie
│   ├── modules.md              # architecture complète des crates (au-delà du MVP)
│   ├── ipc-contract.md         # contrat IPC : événements, verdicts, commandes (aegis-core)
│   ├── detection-catalog.md    # ce qu'on traque par tactique MITRE + signal + moteur
│   ├── policy-model.md         # modes detection/prevention, réponse graduée, faux positifs
│   ├── threat-intel.md         # feeds, mises à jour signées, IOC, réputation de hash
│   ├── ui-spec.md              # design system, écrans, motion, temps réel
│   ├── qa-and-perf.md          # budget perf, bench offensif, corpus, CI, E2E
│   ├── distribution-and-release.md # packaging multi-distro, canaux, signature
│   └── security-and-governance.md  # threat model produit, anti-tamper, gouvernance OSS
├── rules/               # règles YARA + comportementales (vide)
├── scripts/             # start/stop/restart (à créer)
└── logs/                # logs runtime (reset à chaque restart)
```

## Structure prévue (cible)

```
aegis/
├── crates/                      # workspace Cargo (code Rust)
│   ├── aegis-core/              # types partagés, contrat IPC, logging          [v0.1]
│   ├── aegis-probes/            # sondes eBPF + fanotify                         [v0.1]
│   ├── aegis-detection/         # YARA (fichier/mémoire), comportemental, corrélation [v0.1]
│   ├── aegis-response/          # quarantaine, restauration, kill, isolation, rollback [v0.5]
│   ├── aegis-daemon/            # orchestrateur, policy engine, socket + WebSocket [v0.1]
│   ├── aegis-net/               # réseau host : connexions par process, C2, egress [v0.5]
│   ├── aegis-intel/             # feeds, updates signées, store IOC, réputation hash [v1.0]
│   ├── aegis-forensics/         # event store, timeline, threat hunting, export   [v1.0]
│   ├── aegis-cli/               # interface ligne de commande (aegisctl)          [v1.0]
│   ├── aegis-integrity/         # FIM, intégrité binaires système, rootkit        [v1.x]
│   └── aegis-sandbox/           # détonation isolée (bubblewrap/namespaces)        [v1.x]
├── ui/                          # front React/TS/Tailwind + Tauri (client de contrôle)
├── rules/                       # yara/ + behavior/ + ioc/ + SOURCES.md
├── packaging/                   # AUR/.deb/.rpm/Flatpak/AppImage, unit systemd, post-install [v1.0]
├── tests/redteam/               # bench offensif simulé (EICAR, ransomware jouet, reverse shell…)
├── scripts/                     # start.sh / stop.sh / restart.sh (gestion PID, reset logs)
└── logs/                        # logs runtime
```
