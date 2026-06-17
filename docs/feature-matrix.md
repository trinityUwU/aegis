# Feature matrix — Aegis (exhaustive)

Inventaire complet des capacités du produit, sur toute sa durée de vie. Colonne
**Ver.** = version cible (cf. `roadmap-versions.md`). Le périmètre va bien au-delà
du MVP ; tout n'est pas livré d'un coup, mais tout est pensé dès maintenant.

## A. Détection par signatures
| Capacité | Ver. |
|---|---|
| Scan YARA on-demand (fichier, dossier récursif) | v0.1 |
| Scan YARA on-access (déclenché fanotify sur exec) | v0.1 |
| Scan YARA en mémoire (`/proc/<pid>/mem`) — fileless | v0.5 |
| Scan planifié (scheduler) | v1.0 |
| Scan d'archives (zip, tar, gz, 7z…) avec limites de profondeur | v1.x |
| FIM → scan YARA déclenché sur modification de fichier critique | v0.5 |
| Réputation de hash locale (jeux de hash malveillants connus) | v1.0 |
| Détection par patterns hex/MD5 (cascade type LMD) | v1.x |

## B. Détection comportementale (eBPF)
| Capacité | Ver. |
|---|---|
| Exec monitoring (qui lance quoi, d'où, avec quelles caps) | v0.1 |
| Exécution depuis zone suspecte (`/tmp`, `/dev/shm`, world-writable) | v0.5 |
| Fileless : `mmap` anonyme W+X (BPF-LSM `security_mmap_file`) | v0.5 |
| Escalade de privilèges (capset, setuid, transition uid→0) | v0.5 |
| Persistance (cron, systemd, autostart, `ld.so.preload`, rc) | v0.5 |
| Détection de LOLBins (binaires légitimes détournés) | v1.x |
| Injection de process / ptrace abusif | v1.x |
| Corrélation multi-signaux par arbre de pid (faisceau → incident) | v0.5 |
| Baseline d'apprentissage (réduction du bruit initial) | v1.0 |

## C. Anti-ransomware
| Capacité | Ver. |
|---|---|
| Fichiers canari surveillés (fanotify) | v0.5 |
| Détection de rafale (N écritures/renames en T s par process) | v0.5 |
| Analyse d'entropie (détection de chiffrement) | v0.5 |
| Kill in-kernel avant propagation | v0.5 |
| Rollback (restauration via shadow copies / snapshots protégés) | v1.x |

## D. Réseau host
| Capacité | Ver. |
|---|---|
| Surveillance des connexions sortantes par process (eBPF) | v0.5 |
| Détection reverse shell (shell + socket redirigé) | v0.5 |
| Connexion vers IP non résolue par DNS préalable (heuristique C2) | v1.0 |
| Blocage egress par process (mode prevention) | v1.x |
| Détection DNS tunneling / beaconing (heuristique) | v2.0 |

## E. Réponse
| Capacité | Ver. |
|---|---|
| Alerte + notification desktop | v0.1 |
| Quarantaine (isolation + métadonnées + restauration) | v0.5 |
| Kill process (SIGKILL userspace ou in-kernel) | v0.5 |
| Isolation process (cgroup freezer, coupe sockets) | v1.0 |
| Réponse graduée par sévérité et par catégorie | v0.5 |
| Playbooks de réponse configurables | v2.0 |

## F. Forensics & threat hunting
| Capacité | Ver. |
|---|---|
| Event store local (historique d'incidents) | v1.0 |
| Timeline d'incident (séquence + arbre de process) | v1.0 |
| Recherche historique de TTP (hunting sur telemetry passée) | v2.0 |
| Export d'incident (JSON, mapping MITRE) | v1.0 |
| Rapport forensique lisible | v2.0 |

## G. Rootkit & intégrité
| Capacité | Ver. |
|---|---|
| Vérification d'intégrité des binaires système | v1.x |
| Détection de rootkits userland (hooks, fichiers cachés, LD_PRELOAD) | v1.x |
| Détection d'incohérences kernel (modules, hooks cachés) | v2.0 |
| Conscience Secure Boot / mesure d'intégrité au boot | v2.0 |

## H. Sandbox / détonation
| Capacité | Ver. |
|---|---|
| Détonation locale d'un fichier suspect en environnement isolé (bubblewrap/namespaces) | v1.x |
| Capture du comportement (syscalls, fichiers, réseau) en sandbox | v2.0 |

## I. Threat intelligence
| Capacité | Ver. |
|---|---|
| Bundle de règles YARA embarqué | v0.1 |
| Mise à jour des règles (déclenchée/planifiée, signée) | v1.0 |
| Agrégation de feeds open source (abuse.ch, Neo23x0, Elastic…) | v1.0 |
| Import d'IOC (hash, domaines, IP) | v1.x |
| Bundle de mise à jour hors-ligne | v1.0 |

## J. Device control
| Capacité | Ver. |
|---|---|
| Détection d'exécution depuis média amovible (USB) | v1.x |
| Politique de blocage d'exécution sur média amovible | v2.0 |
| Scan automatique au montage USB | v1.x |

## K. Protection des téléchargements (host-side)
| Capacité | Ver. |
|---|---|
| Scan automatique des fichiers écrits dans `~/Downloads` | v1.0 |
| Marquage des fichiers issus du web (attr xdg origin) | v2.0 |

## L. Maintenance & scheduler
| Capacité | Ver. |
|---|---|
| Scans planifiés | v1.0 |
| Mises à jour de règles planifiées | v1.0 |
| Rotation/archivage de l'historique | v1.0 |

## M. Interface
| Capacité | Ver. |
|---|---|
| Dashboard temps réel (état, flux live) | v0.1 |
| Vue détections/incidents (sévérité, MITRE, arbre process) | v0.5 |
| Gestion de la quarantaine | v0.5 |
| Gestion des règles & policies | v1.0 |
| Vue forensics/timeline | v1.0 |
| Paramètres (modes, exclusions, updates) | v0.5 |
| Tray + notifications natives | v0.5 |
| CLI complète (`aegisctl`) | v1.0 |
| Webapp via WebSocket local | v1.0 |

## N. Policy & modes
| Capacité | Ver. |
|---|---|
| Mode detection / prevention (global) | v0.5 |
| Mode par catégorie de menace | v0.5 |
| Exclusions (chemin, hash, process) | v0.5 |
| Profils de policy (préréglages) | v2.0 |
| Boucle de feedback faux positifs | v1.0 |

## O. Gestion & extensibilité
| Capacité | Ver. |
|---|---|
| API locale (socket Unix + WebSocket) | v0.5 |
| Plugins de règles tiers | v2.0 |
| Fleet multi-host local (console centrale optionnelle) | v3.0 |
| API externe opt-in (jamais par défaut) | v3.0 |

## P. Système & distribution
| Capacité | Ver. |
|---|---|
| Service systemd, démarrage au boot | v0.5 |
| Capabilities minimales (pas de root nu) | v1.0 |
| Auto-protection du daemon et des sondes | v1.0 |
| Packaging AUR / .deb / .rpm / Flatpak / AppImage | v1.0 |
| Canaux nightly / beta / stable | v1.0 |

## Q. Anti-évasion & rootkits nouvelle génération (l'avantage concurrentiel)
Détail et justification : [`competitive-edge.md`](./competitive-edge.md).
| Capacité | Ver. |
|---|---|
| Deception / honeypots (fausses clés SSH, honey tokens, leurres) | v1.0 |
| Decloak cross-view des process cachés (`/proc` vs bruteforce PID vs eBPF) | v1.0 |
| Decloak cross-view des fichiers cachés (`getdents`) | v1.x |
| Détection des rootkits eBPF (audit programmes chargés + hooks suspects) | v1.x |
| Baseline des programmes eBPF légitimes | v1.x |
| Drift detection (binaires, users, SSH keys, cron/systemd, modules, taint) | v1.x |
| Intégrité kernel & anti-masquerading LKM (taint, kallsyms, modules) | v2.0 |
| Monitoring io_uring (bypass syscall) | v2.0 |
| Auto-protection anti-subversion des sondes Aegis | v1.0 |
| Decloak eBPF par lecture mémoire kernel (`prog_idr`) | v2.0 |

## Explicitement hors scope (cf. product-vision)
VPN · gestionnaire de mots de passe · contrôle parental · webcam guard · NIDS
périmétrique · SIEM/XDR cloud · scanner de vulnérabilités de paquets.
