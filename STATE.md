# STATE — Aegis

> Résumé vivant cross-session. Garder < 300 lignes.

## Statut global

**Phase : conception / documentation.** Aucun code applicatif. Le dépôt contient
la vision, l'architecture, la roadmap et la structure de docs. Prochaine étape
exécutable : scaffold du workspace Cargo et du projet UI (Phase 0, non démarré).

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

- Nom de travail : **Aegis** (à confirmer).
- Repo local : `/mnt/projects/aegis`. Repo GitHub public à créer après confirmation.
