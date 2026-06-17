# Catalogue de détection — Aegis

Ce que Aegis traque sur un host Linux, par tactique MITRE ATT&CK, avec le signal
capté et le moteur responsable. Transforme « comportemental » en spec concrète.
Chaque entrée = un `ThreatCategory` + technique(s) MITRE + source du signal.

Légende source : **EX** eBPF exec · **FA** fanotify · **MM** eBPF mmap-lsm ·
**PR** eBPF priv · **NET** eBPF socket · **YR** YARA · **FIM** intégrité fichiers.

## Execution (TA0002)
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Exécution depuis `/tmp`, `/dev/shm`, dossier world-writable | T1059 | EX+FA | High |
| Interpréteur lancé avec script inline (`bash -c`, `python -c …`) suspect | T1059.004/.006 | EX | Medium |
| Binaire fraîchement écrit puis exécuté (< quelques s) | T1059 | FA+EX | High |
| Exécution mémoire : `mmap` anonyme W+X puis saut (fileless/shellcode) | T1620 | MM | Critical |

## Persistence (TA0003)
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Ajout/modif tâche cron | T1053.003 | FA | Medium |
| Création/modif unité systemd (user/system) | T1543.002 | FA | Medium |
| Modif `~/.bashrc`, `~/.profile`, `~/.config/autostart` | T1546.004 | FA | Medium |
| Injection `LD_PRELOAD` / `/etc/ld.so.preload` | T1574.006 | FA+EX | High |

## Privilege Escalation (TA0004)
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Élévation de capabilities inattendue (`capset`) | T1548 | PR | High |
| Exécution d'un binaire setuid hors liste connue | T1548.001 | EX+PR | Medium |
| Transition uid → 0 par un process non autorisé | T1068 | PR | Critical |

## Defense Evasion (TA0005)
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Suppression/troncature de logs (`/var/log`) | T1070.002 | FA | High |
| Masquerading : `comm` ≠ exe réel, nom de process trompeur | T1036 | EX | Medium |
| Tentative d'arrêt/altération du daemon Aegis lui-même | T1562.001 | EX+PR | Critical |

## Credential Access (TA0006)
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Lecture `/etc/shadow` par process non légitime | T1003.008 | FA | High |
| Accès aux clés SSH privées (`~/.ssh/id_*`) | T1552.004 | FA | High |
| Lecture `/proc/<pid>/maps`/`mem` d'un process tiers | T1003 | EX | High |

## Command & Control (TA0011)
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Reverse shell : shell avec stdin/stdout redirigés vers socket (`dup2`) | T1059 | EX+NET | Critical |
| Connexion sortante vers IP non résolue par DNS préalable | T1071 | NET | Medium |
| Bénéficiaire d'une connexion = process depuis dossier temporaire | T1105 | NET+EX | High |

## Impact (TA0040) — anti-ransomware
| Détection | MITRE | Source | Sévérité |
|---|---|---|---|
| Écriture/rename/unlink sur **fichier canari** | T1486 | FA | Critical |
| Rafale : N fichiers réécrits/renommés en T secondes par un même process | T1486 | FA | Critical |
| Hausse d'entropie marquée sur fichiers réécrits (chiffrement) | T1486 | FA+YR | Critical |
| Suppression des copies/sauvegardes locales | T1490 | FA | High |

## Signatures (transverse)
| Détection | Source | Sévérité |
|---|---|---|
| Match YARA sur fichier (on-access / on-demand / planifié) | YR | selon règle |
| Match YARA en mémoire d'un process (payload jamais sur disque) | YR | High–Critical |
| Modification d'un fichier critique surveillé → scan YARA déclenché | FIM+YR | selon règle |

## Stratégie anti-faux-positifs
- Baseline d'apprentissage au premier déploiement (process/chemins habituels).
- Allowlist par hash/chemin/process, exclusions explicites.
- Confiance pondérée : une règle seule alerte ; corrélation multi-signaux
  (ex: exec /tmp **+** connexion sortante **+** capset) → escalade automatique.
