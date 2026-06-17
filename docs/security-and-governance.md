# Modèle de sécurité & gouvernance — Aegis

Un antivirus est une cible de choix (privilèges élevés, position centrale) et un
acte de confiance. Sa propre sécurité et sa gouvernance sont des features, pas un
détail.

## Threat model du produit lui-même

Aegis tourne en privilégié et voit tout : sa compromission est catastrophique.
Surfaces et mitigations :

| Surface | Risque | Mitigation |
|---|---|---|
| **Parsing de fichiers hostiles** (YARA sur fichier/archive) | exploit du parser → RCE dans le daemon | `yara-x` en Rust (mémoire sûre), limites de taille/profondeur d'archive, parsing isolé |
| **Socket Unix / WebSocket local** | process local non autorisé pilote le daemon | permissions groupe `aegis`, pas de port réseau, validation stricte des commandes |
| **Pipeline de mise à jour** | injection de règles malveillantes | vérification de signature obligatoire, rejet si non vérifié (cf. `threat-intel.md`) |
| **Sondes eBPF / fanotify** | désactivation par un attaquant (T1562) | auto-protection : toute tentative d'arrêt/altération du daemon ou des sondes = détection **Critical** |
| **Élévation via le daemon** | abus des capabilities | **moindre privilège** : capabilities ciblées, pas root nu ; durcissement systemd |
| **Déni de service** | event flood → saturation CPU | filtrage in-kernel, backpressure, rate-limiting des events remontés |
| **Rootkit kernel** | subversion d'eBPF/LSM eux-mêmes | limite assumée et documentée ; conscience Secure Boot (v2.0) pour réduire la fenêtre |

## Auto-protection (anti-tamper)
- Le daemon surveille sa propre intégrité et celle de ses sondes.
- Tentative de kill/stop/unload par un tiers non autorisé → incident Critical +
  réaction (re-chargement des sondes, alerte).
- Fichiers de quarantaine et store forensique protégés en écriture.

## Vie privée (non négociable)
- **Zéro télémétrie par défaut.** Aegis n'émet aucune donnée vers l'extérieur.
- Tout l'historique, les incidents, les hashes restent **locaux**.
- Les mises à jour sont des **pulls** déclenchés par l'utilisateur ; rien ne part
  dans l'autre sens. Argument RGPD et souveraineté natif.

## Gouvernance open source

| Fichier | Rôle | Ver. |
|---|---|---|
| `LICENSE` | MIT (→ licence Echo ultérieurement) | v0.1 ✅ |
| `SECURITY.md` | politique de divulgation des vulnérabilités (responsible disclosure) | v0.1 |
| `CONTRIBUTING.md` | standards de contribution, build, lint, tests | v1.0 |
| `CODE_OF_CONDUCT.md` | règles de communauté | v1.0 |
| `CHANGELOG.md` | journal des versions | v1.0 |

- **Divulgation responsable** : un canal de contact sécurisé, délai de correction
  avant publication, pas de honte du bug — on est un projet de sécurité, on traite
  nos propres failles avec la rigueur qu'on applique aux menaces.
- **Auditabilité** : le code reste lisible et l'objectif de reproducible builds
  (cf. `distribution-and-release.md`) permet de vérifier que le binaire = le source.
- **Pas de fonctionnalité cachée** : tout ce que le daemon fait est documenté ;
  un AV opaque ne mérite pas la confiance qu'on lui demande.

## Licence Echo (transition future)
Le passage de MIT à la licence Echo (quand elle sera définie) devra préserver le
caractère auditable et la liberté d'usage local pour l'utilisateur — à arbitrer
avec Chris le moment venu, sans rétroactivité sur les versions déjà publiées en MIT.
