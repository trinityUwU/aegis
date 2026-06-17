# Spécification UI — Aegis

L'interface est l'angle d'attaque du produit : la sécurité runtime n'a jamais eu
de visage clair en open source. Référence esthétique Apple + Anthropic — dark,
épuré, cohérent, calme. Le rendu visuel et le motion sont validés par Chris.

## Identité visuelle

- **Thème dark obligatoire**, fond profond non noir pur (évite l'écrasement OLED).
- **Palette sémantique** — chaque couleur a une raison, jamais décorative :
  - `safe` (vert calme) — état protégé, aucun incident.
  - `info` (bleu) — activité normale, événements.
  - `warning` (ambre) — sévérité low/medium, attention.
  - `critical` (rouge maîtrisé) — high/critical, action requise.
  - `neutral` (gris) — chrome, texte secondaire.
  La couleur ne hurle pas : un dashboard sain est calme et sombre, pas un sapin de
  Noël d'alertes.
- **Typographie distinctive** — pas Inter, pas Roboto. Une grotesque technique
  lisible (ex. famille type Geist / Söhne-like) + mono pour les chemins/hashes.
- **Système modulaire parent/enfant** : tokens hérités, override par type d'écran.
- Palette définie par l'identité du projet, pas de palette globale hardcodée.

## Architecture des écrans

| Écran | Rôle | Ver. |
|---|---|---|
| **Overview** | État de protection global, santé du daemon, compteurs, dernières détections. La première chose qu'on voit : « suis-je protégé ? » | v0.1 |
| **Live activity** | Flux temps réel des événements (exec, accès, réseau), filtrable, calme par défaut. | v0.1 |
| **Detections / Incidents** | Liste des verdicts : sévérité, catégorie, **mapping MITRE**, arbre de process, chronologie. Drill-down par incident. | v0.5 |
| **Quarantine** | Fichiers isolés, métadonnées, restauration, suppression définitive. | v0.5 |
| **Scan** | Scan on-demand (chemin, dossier), scan mémoire, résultats. | v0.5 |
| **Rules & Policy** | Jeux de règles YARA actifs, règles comportementales, modes par catégorie, exclusions. | v1.0 |
| **Forensics / Timeline** | Historique d'incidents, timeline, threat hunting, export. | v1.0 |
| **Settings** | Modes detection/prevention, zones surveillées, updates, planification, baseline. | v0.5 |

## Composants transverses

- **Tray** : icône d'état (safe/warning/critical), accès rapide, mode actif.
- **Notifications natives** : sévérité ≥ medium, actionnables (voir / quarantaine).
- **Severity badge**, **MITRE tag**, **process tree** (arbre interactif),
  **event row** (densité maîtrisée), **empty states** soignés (un état vide n'est
  pas un bug, c'est « tout va bien »).
- **Mode switch** detection ⇄ prevention, visible et explicite.

## Motion (conforme aux standards)

- **Framer Motion** : transitions d'écran, apparition des événements dans le flux
  (entrée douce, jamais de clignotement anxiogène), layout animations.
- **GSAP/ScrollTrigger** réservé si une vue de storytelling/onboarding le justifie.
- Le motion sert la lisibilité (suivre un nouvel événement, comprendre une
  corrélation), jamais la démonstration d'effet. Respect du budget perf côté UI.

## Connexion temps réel

- L'UI est un **client** : se connecte au daemon via WebSocket local (JSON).
- Reçoit en flux : événements (filtrés), verdicts, état. Envoie : commandes
  (quarantaine, kill, mode, exclusions). Reconnexion transparente si le daemon
  redémarre — l'UI ne porte aucun état de protection.
- Le même front est servable en webapp (navigateur) sans code supplémentaire.

## CLI (`aegisctl`) — interface jumelle
Toute action UI a un équivalent CLI (status, scan, quarantine, mode, intel update,
export). Pour les serveurs headless et l'automatisation locale. Ver. v1.0.
