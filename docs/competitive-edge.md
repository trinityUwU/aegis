# Avantage concurrentiel — Aegis

Les features de base (scan signature, comportemental, quarantaine) sont dans la
`feature-matrix`. Ce document liste ce qui fait qu'Aegis **bat les concurrents
sérieux** : les angles morts de l'industrie en 2026, et la position que personne
n'occupe.

## Le champ de bataille réel (2026)

La menace Linux de pointe n'est plus le malware sur disque : ce sont les
**rootkits eBPF et io_uring** (TripleCross, Boopkit, RingReaper, **VoidLink** —
documenté par Check Point en janvier 2026). Ils :
- hookent `execve`/`getdents` pour la persistance et le masquage,
- interceptent `sys_bpf` pour **se cacher de `bpftool`** (l'outil de détection
  lui-même est aveuglé),
- contournent les hooks syscall via **`io_uring`** (I/O sans syscalls hookés),
- sont **invisibles à rkhunter/chkrootkit** (pas de LKM out-of-tree),
- **aveuglent les EDR agent-based** qui tournent dans le kernel compromis.

Conclusion : la plupart des produits voient ce que le rootkit veut bien leur
montrer. C'est l'opportunité.

## Les 8 features qui font la différence

### 1. Détection des rootkits eBPF eux-mêmes
Audit des programmes eBPF chargés et de leurs points d'attache. Sur un desktop,
un programme eBPF inattendu attaché à un tracepoint, un kprobe (`execve`,
`getdents`, `sys_bpf`, `tcp_*`) ou un hook LSM est un indicateur fort de
compromission. **Baseline** des programmes légitimes (les nôtres + ceux du système
connus) → tout le reste est suspect. Quasi aucun AV grand public ne couvre ça.

### 2. Anti-evasion par cross-view (le cœur)
**Ne jamais faire confiance à une source unique.** Croiser systématiquement :
- liste `/proc` vs bruteforce de PID (accès direct) vs vue kernel eBPF → process caché,
- `getdents` userspace vs lecture directe → fichiers cachés,
- `BPF_PROG_GET_NEXT_ID` vs état kernel → programmes eBPF cachés.
Toute divergence entre deux vues qui devraient concorder = decloaking d'une
entité dissimulée. Approche Sandfly, mais **en temps réel continu**, pas en scan
ponctuel.

### 3. Monitoring io_uring
L'angle mort des EDR per-syscall : `io_uring` exécute des opérations
fichier/réseau **sans passer par les syscalls hookés**. On surveille
`io_uring_setup`/`enter`/`register`, les batchs d'opérations anormalement gros et
l'enregistrement excessif de descripteurs — signaux d'une collecte/exfiltration
furtive.

### 4. Drift detection
Surveillance continue des composants Linux les plus ciblés, avec baseline immuable
et alerte sur déviation :
- nouveau binaire exécuté (sur disque **ou fileless**),
- nouvel utilisateur, nouvelle **clé SSH**, changement de login shell,
- nouvelle tâche `cron`/`at`/`systemd`,
- **nouveau module kernel chargé / changement de kernel taint**.

### 5. Intégrité kernel & anti-masquerading LKM
Surveillance des taint flags, des modules chargés, de `/proc/kallsyms` (hooks).
Détection du masquerading (VoidLink se fait passer pour `amd_mem_encrypt`) :
un module au nom légitime mais non signé / non attendu = alerte.

### 6. Deception / honeypots
Extension des canaris ransomware en système de leurres : **fausses clés SSH, faux
credentials, honey tokens** placés dans des emplacements qu'aucun process légitime
ne touche. Tout accès = intrusion à très haute confiance, **quasi zéro faux
positif** — le meilleur ratio signal/bruit de tout l'arsenal.

### 7. Auto-protection anti-subversion
Aegis vérifie que **ses propres sondes** ne sont pas détachées, hookées ou
aveuglées. Un rootkit qui tente de neutraliser Aegis déclenche lui-même une
détection Critical.

### 8. Decloak par lecture mémoire kernel (cas avancés)
Quand le syscall est suspecté filtré, énumérer depuis l'état kernel (`prog_idr`)
plutôt que via `bpf(BPF_PROG_GET_NEXT_ID)` qui peut être manipulé. Dernier recours
forensique pour les rootkits eBPF furtifs.

## Tableau comparatif — qui rate quoi

| Capacité | rkhunter/chkrootkit | Falco/Tetragon | EDR propriétaire (CS/S1) | Sandfly | **Aegis** |
|---|---|---|---|---|---|
| Comportemental temps réel | ❌ | ✅ | ✅ | partiel (scan) | ✅ |
| Détection rootkit eBPF | ❌ | ❌ | partiel | ✅ | ✅ |
| Decloak cross-view | ❌ | ❌ | partiel | ✅ | ✅ |
| Monitoring io_uring | ❌ | partiel | partiel | partiel | ✅ |
| Drift detection | ❌ | ❌ | partiel | ✅ | ✅ |
| Deception/honeypots | ❌ | ❌ | partiel | partiel | ✅ |
| Signatures fichier/mémoire | partiel | ❌ | ✅ | ✅ | ✅ |
| 100 % local, zéro télémétrie | ✅ | ✅ | ❌ | ❌ (SSH/cloud) | ✅ |
| UI desktop soignée | ❌ | ❌ | ✅ | partiel | ✅ |
| Open source / auditable | ✅ | ✅ | ❌ | ❌ | ✅ |
| Gratuit | ✅ | ✅ | ❌ | ❌ | ✅ |

## La position que personne n'occupe

Falco/Tetragon = temps réel mais pas de chasse aux rootkits ni decloak.
Sandfly = decloak/drift/hunting excellents mais agentless (scan ponctuel, pas de
prévention temps réel continue), propriétaire et cloud-orchestré.
EDR US = complets mais fermés, cloud, télémétrie, payants, aveuglables.

**Aegis est le seul à réunir : temps réel continu + anti-evasion (decloak, eBPF
rootkit, io_uring, drift) + deception + signatures + 100 % local + open source +
beau.** C'est la thèse produit.

## Limite honnête
Aegis est lui-même agent-based eBPF : un rootkit ring-0 parfait peut toujours
mentir à n'importe quel agent. Notre défense n'est pas une immunité magique mais
le **cross-view systématique** (jamais une source unique de vérité) + l'auto-
protection, qui élèvent radicalement le coût de l'évasion. On documente cette
limite plutôt que de la masquer (cf. `security-and-governance.md`).
