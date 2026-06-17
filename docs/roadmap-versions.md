# Roadmap par versions — Aegis

Versioning **SemVer**. Trois canaux : `nightly` (build auto), `beta` (préversions
testables), `stable` (releases signées). Chaque version a un thème, un contenu, un
public et des **critères de sortie** (definition of done) qui conditionnent le
passage à la suivante. Détail des features : `feature-matrix.md`.

---

## v0.1 → v0.4 — Alpha « Le squelette qui voit »
**Thème** : capter en temps réel et détecter sur signature.
**Contenu** : fondation (workspace, IPC, scripts), capteurs eBPF exec + fanotify,
daemon d'ingestion, yara-x on-demand/on-access, UI affichant le flux live.
(= Lots 0→2 + amorce Lot 4 du plan d'exécution.)
**Public** : Chris, dev interne.
**Sortie** : le daemon tourne en continu, capte les exec/accès en temps réel,
détecte EICAR et met en quarantaine, l'UI affiche le flux. Zéro crash, budget CPU
au repos tenu.

## v0.5 → v0.9 — Beta « Le système qui agit »
**Thème** : comportemental, anti-ransomware, réponse, UI complète.
**Contenu** : sondes mmap/capset/persistance/réseau, corrélation par arbre de pid,
anti-ransomware (canaris + rafale + entropie + kill), réponse graduée, modes
detection/prevention par catégorie, scan mémoire, UI détections + quarantaine +
settings + tray. Service systemd.
**Public** : early adopters Linux avertis.
**Sortie** : ransomware simulé tué avant propagation, reverse shell détecté,
corrélation multi-signaux fonctionnelle, faux positifs sous contrôle en mode
detection, UI opérationnelle.

## v1.0 — Stable « L'antivirus qu'on installe »
**Thème** : production-ready, distribuable, maîtrisé.
**Contenu** : threat intel (feeds open source + updates signées + bundle
offline), forensics (event store + timeline + export), baseline d'apprentissage,
isolation process, réputation de hash, scan planifié + downloads, CLI `aegisctl`,
capabilities minimales, auto-protection, packaging multi-distro, doc utilisateur.
**Public** : tout utilisateur Linux desktop.
**Sortie** : un tiers installe via son gestionnaire de paquets et l'utilise au
quotidien sans intervention dev ; taux de faux positifs mesuré et acceptable ;
benchmarks perf publiés ; release signée.

## v1.x — « La profondeur »
**Thème** : couvrir les vecteurs avancés.
**Contenu** : détection rootkit userland + intégrité des binaires système,
device control USB + scan au montage, sandbox de détonation (bubblewrap),
rollback ransomware (snapshots), scan d'archives, détection LOLBins + injection,
import d'IOC, blocage egress par process.
**Sortie** : couverture de la matrice étendue aux menaces avancées, sandbox
fonctionnelle, rollback démontré.

## v2.0 — « Le chasseur »
**Thème** : investigation et personnalisation expertes.
**Contenu** : threat hunting historique (recherche TTP sur telemetry passée),
rapports forensiques, profils de policy, playbooks de réponse, plugins de règles
tiers, incohérences kernel + conscience Secure Boot, beaconing/DNS tunneling,
politique de blocage device.
**Public** : utilisateurs avancés, équipes sécurité souveraines.
**Sortie** : un analyste peut enquêter sur un incident passé de bout en bout sans
outil externe.

## v3.0 — « Au-delà du poste »
**Thème** : extension contrôlée, dans le respect des invariants.
**Contenu** : fleet multi-host **local** (console centrale optionnelle, pas de
cloud imposé), API externe **opt-in** strictement désactivée par défaut,
exploration d'une couche heuristique avancée **CPU-only** (jamais de VRAM, jamais
au prix du budget perf — décision conditionnée à un gain réel mesuré).
**Sortie** : plusieurs postes supervisables depuis une instance locale sans rien
envoyer hors du réseau de l'utilisateur.

---

## Règles de version
- Une feature ne « descend » jamais de version : si elle n'est pas prête, elle
  glisse à la suivante, elle ne dégrade pas le critère de sortie de la courante.
- Pas de v_n+1 tant que les critères de sortie de v_n ne sont pas remplis.
- Les invariants (local-first, zéro télémétrie, perf, zéro VRAM) ne sont jamais
  négociés pour livrer une feature — une feature qui les viole est rejetée.
