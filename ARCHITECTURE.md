# Architecture — Aegis

## Principe directeur

Un **daemon privilégié et persistant** fait tout le travail (capteurs kernel,
détection, réponse). L'**UI est un simple client de contrôle** qui se connecte au
daemon ; ouvrir ou fermer la fenêtre n'interrompt jamais la protection. Le daemon
n'ouvre **aucun port réseau** — toute communication passe par un socket Unix local.

```
┌─ UI (React/TS, Tauri) — client de contrôle ───────────────┐
│   WebSocket local  ←  événements, verdicts, état           │
│                    →  commandes (quarantaine, kill, toggle)│
├─ Daemon Rust (le cerveau, systemd, privilégié) ────────────┤
│   policy engine · corrélation MITRE · état · socket Unix   │
├─ Couches métier (crates Rust) ─────────────────────────────┤
│   detection · response                                     │
├─ Capteurs kernel ──────────────────────────────────────────┤
│   fanotify (blocage fichiers) + eBPF/BPF-LSM (comportement)│
└─────────────────────────────────────────────────────────────┘
```

## Découpage par domaine (Screaming Architecture)

Workspace Cargo, un crate par domaine de sécurité. La structure crie ce que le
produit **fait**, pas ce avec quoi il est bâti.

| Domaine (`crates/`) | Responsabilité unique — définition non ambiguë |
|---|---|
| `aegis-probes` | **Capteurs bas niveau.** Charge et pilote les sondes eBPF et fanotify. Transforme les événements kernel bruts en événements typés. Ne décide rien. |
| `aegis-detection` | **Verdict.** Reçoit les événements, applique les moteurs (YARA fichier/mémoire, règles comportementales, anti-ransomware). Produit un verdict + mapping MITRE. N'agit pas. |
| `aegis-response` | **Action.** Exécute la réponse (quarantaine, restauration, kill process, isolation). Ne détecte rien. |
| `aegis-daemon` | **Orchestration.** Assemble probes → detection → response, tient l'état, le policy engine, expose le socket Unix + le bridge WebSocket. Point d'entrée. |
| `aegis-core` | **Partagé réel uniquement.** Types communs, contrat IPC, logging. Aucune logique métier. |

UI hors workspace Rust :

| Dossier | Responsabilité |
|---|---|
| `ui/` | Front React/TS/Tailwind + intégration Tauri. Présentation et contrôle uniquement, zéro logique de détection. |
| `rules/` | Règles YARA et règles comportementales, versionnées et mises à jour indépendamment du code. |

## Frontières de modules (Modular Monolith)

- Le flux est **unidirectionnel** : `probes → detection → response`, orchestré par
  `daemon`. Aucun crate métier n'en appelle un autre directement.
- `aegis-core` ne dépend de personne ; tout le monde peut en dépendre.
- Un domaine ne touche jamais les internes d'un autre : tout passe par les types
  publics exposés dans `aegis-core` (contrat IPC) ou l'interface publique du crate.
- **Anti-rot** : `probes` ne décide pas, `detection` n'agit pas, `response` ne
  détecte pas. Si l'une de ces frontières se brouille, c'est un red flag.

## Frontière de sécurité

- Daemon = privilèges élevés (capacités kernel, fanotify, eBPF). Surface minimale,
  audité en priorité.
- UI = aucun privilège. Communique uniquement via le socket local.
- Pas de webview lourde embarquée (Tauri, pas Electron) pour réduire la surface
  d'attaque du composant le plus exposé visuellement.

## DRY

- Source de vérité unique pour le contrat d'événement/verdict : `aegis-core`.
- Duplication tolérée entre règles YARA et logique comportementale si elles
  évoluent pour des raisons différentes — pas de mutualisation prématurée.
