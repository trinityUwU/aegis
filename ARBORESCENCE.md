# Arborescence — Aegis

Une ligne par fichier. Le code applicatif n'existe pas encore : la section
**Existant** liste le présent, **Structure prévue** documente la cible Phase 0+.

## Existant

```
aegis/
├── LICENSE              # licence MIT
├── README.md            # présentation, stack, principes
├── ARCHITECTURE.md      # domaines, frontières de modules, sécurité
├── STATE.md             # état vivant cross-session
├── TODO.md              # roadmap MVP + backlog
├── ARBORESCENCE.md      # ce fichier
├── .echoforge.yml       # métadonnées projet EchoForge
├── .gitignore           # exclusions git
├── .env.example         # variables d'environnement (vide de secrets)
├── docs/                # documentation détaillée (vide)
├── rules/               # règles YARA + comportementales (vide)
├── scripts/             # start/stop/restart (à créer)
└── logs/                # logs runtime (reset à chaque restart)
```

## Structure prévue (cible)

```
aegis/
├── crates/                      # workspace Cargo (code Rust)
│   ├── aegis-core/              # types partagés, contrat IPC, logging
│   ├── aegis-probes/            # sondes eBPF + fanotify
│   ├── aegis-detection/         # moteurs YARA (fichier/mémoire), comportemental, anti-ransomware
│   ├── aegis-response/          # quarantaine, restauration, kill, isolation
│   └── aegis-daemon/            # orchestrateur, policy engine, socket Unix + WebSocket
├── ui/                          # front React/TS/Tailwind + Tauri (client de contrôle)
├── rules/                       # règles YARA + règles comportementales versionnées
├── scripts/                     # start.sh / stop.sh / restart.sh (gestion PID, reset logs)
└── logs/                        # logs runtime
```
