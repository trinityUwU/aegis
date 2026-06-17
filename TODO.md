# TODO — Aegis

## Roadmap MVP

Ordre de bataille : fondation → capteurs → détection → comportement → UI →
réponse. Chaque phase a un livrable validé avant d'enchaîner.

### Phase 0 — Fondation
- [x] Conception : vision, cibles, stack, architecture
- [x] Documentation racine (README, ARCHITECTURE, STATE, TODO, ARBORESCENCE)
- [x] Licence MIT
- [x] Dépôt local + git
- [x] Repo GitHub public — https://github.com/trinityUwU/aegis
- [ ] Scaffold workspace Cargo (`crates/`) + projet UI (`ui/`)
- [ ] Scripts `start.sh` / `stop.sh` / `restart.sh` avec gestion PID + reset logs

### Phase 1 — Capteurs kernel + daemon cœur
- [ ] fanotify on-access bloquant sur exécution + zones chaudes (`~/Downloads`, `/tmp`, autostart, cron/systemd user)
- [ ] Sonde eBPF exec-monitoring (`execve` / `bprm_check`) avec contexte (caps, cgroup)
- [ ] Daemon orchestrateur : ingestion événements, socket Unix local
- [ ] Livrable : flux temps réel des exec/accès en logs

### Phase 2 — Moteur de détection signature
- [ ] Intégration YARA natif (`yara-rust`)
- [ ] Scan on-access (déclenché par fanotify), on-demand, planifié
- [ ] Scan mémoire (YARA sur process en RAM) → fileless
- [ ] Quarantaine : isolation + métadonnées + restauration
- [ ] Livrable : EICAR + règle YARA détecté et mis en quarantaine en temps réel

### Phase 3 — Détection comportementale
- [ ] Règles eBPF : escalade privilèges, reverse shell, persistance (cron/systemd/LD_PRELOAD)
- [ ] Anti-ransomware : fichiers canari + détection rafale chiffrement/rename → kill in-kernel
- [ ] Mapping MITRE ATT&CK sur chaque détection
- [ ] Livrable : ransomware simulé tué avant propagation

### Phase 4 — UI temps réel
- [ ] Bridge WebSocket daemon ↔ UI
- [ ] Dashboard React dark épuré : flux live, état protection, détections, quarantaine
- [ ] Contrôles UI→daemon : quarantaine/restauration, exclusions, kill, toggle détection ⇄ prévention
- [ ] Empaquetage Tauri (tray, notifications natives)
- [ ] Livrable : render validé par Chris

### Phase 5 — Réponse & finition
- [ ] Réponse graduée (alerte → isolation → kill)
- [ ] FIM léger sur fichiers critiques (déclenche YARA)
- [ ] Mise à jour des règles YARA (feed open source + règles maison)
- [ ] Livrable : MVP utilisable au quotidien

## Backlog (hors MVP)
- IA locale comportementale (classification séquences syscalls)
- ClamAV optionnel en process séparé (moteur signature additionnel)
- Détection rootkit avancée
- Rollback ransomware (restauration fichiers chiffrés)
- API externe optionnelle + console multi-postes (cas flotte/entreprise)
- Version serveur/VPS
