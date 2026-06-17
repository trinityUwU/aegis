# Modules — architecture complète Aegis

Au-delà des 5 crates du MVP, voici l'architecture cible du produit complet.
Workspace Cargo, un crate par domaine de sécurité (Screaming Architecture).
Colonne **Ver.** = version d'introduction du module.

## Crates Rust

| Crate | Responsabilité (définition non ambiguë) | Ver. |
|---|---|---|
| `aegis-core` | Types partagés, contrat IPC, logging. Aucune logique métier. | v0.1 |
| `aegis-probes` | Capteurs bas niveau : charge/pilote eBPF + fanotify, normalise les événements kernel. Ne décide rien. | v0.1 |
| `aegis-detection` | Verdict : moteurs YARA (fichier/mémoire), règles comportementales, corrélation. Produit un verdict + MITRE. N'agit pas. | v0.1 |
| `aegis-response` | Action : quarantaine, restauration, kill, isolation, rollback. Ne détecte rien. | v0.5 |
| `aegis-daemon` | Orchestration : assemble le pipeline, état, policy engine, socket Unix + bridge WebSocket. Point d'entrée. | v0.1 |
| `aegis-intel` | Threat intelligence : feeds, mises à jour signées, store d'IOC, réputation de hash. | v1.0 |
| `aegis-forensics` | Investigation : event store local, timeline, requêtes de threat hunting, export. | v1.0 |
| `aegis-net` | Réseau host : surveillance des connexions par process, heuristiques C2, blocage egress. | v0.5 |
| `aegis-integrity` | Intégrité & rootkit : FIM, vérification des binaires système, détection rootkit/incohérences kernel. | v1.x |
| `aegis-sandbox` | Détonation isolée (bubblewrap/namespaces), capture comportementale. | v1.x |
| `aegis-cli` | Interface ligne de commande (`aegisctl`) cliente du daemon. | v1.0 |

## Hors workspace Rust

| Composant | Rôle | Ver. |
|---|---|---|
| `ui/` | Front React/TS/Tailwind + Tauri v2. Présentation et contrôle, zéro logique de détection. | v0.1 |
| `rules/` | Règles YARA + règles comportementales, versionnées et mises à jour indépendamment du code. | v0.1 |
| `packaging/` | Recettes AUR/.deb/.rpm/Flatpak/AppImage, unit systemd, scripts post-install. | v1.0 |

## Flux & frontières

```
                         aegis-daemon (orchestrateur)
                                  │
   ┌──────────┬───────────┬───────┼────────┬────────────┬──────────┐
   ▼          ▼           ▼       ▼        ▼            ▼          ▼
aegis-probes  aegis-net  aegis-detection  aegis-response  aegis-intel  aegis-forensics
   │          │           ▲       │                ▲           ▲
   └──events──┴───────────┘   verdicts          updates    persistance
                                  │
                           aegis-integrity, aegis-sandbox (modules sollicités à la demande)
```

Règles de frontière (Modular Monolith) :
- Flux unidirectionnel `probes/net → detection → response`, orchestré par `daemon`.
  Aucun crate métier n'en appelle un autre directement.
- `aegis-core` ne dépend de personne ; tout le monde peut en dépendre.
- Tout passe par les types publics de `aegis-core` (contrat IPC), jamais par les
  internes d'un autre crate.
- `intel`, `forensics`, `integrity`, `sandbox`, `cli` sont des modules
  périphériques branchés sur le daemon ; on peut en extraire un en service
  séparé plus tard sans réécriture.
- Anti-rot (cf. `ARCHITECTURE.md`) : `probes` ne décide pas · `detection` n'agit
  pas · `response` ne détecte pas · `intel` ne scanne pas · `forensics` ne juge pas.

## Trajectoire de découpage
- **v0.x** : `core`, `probes`, `detection`, `response`, `daemon`, `net`, `ui`, `rules`.
- **v1.0** : ajout `intel`, `forensics`, `cli`, `packaging`.
- **v1.x → v2.0** : ajout `integrity`, `sandbox` ; enrichissement `forensics`.
- **v3.0** : extraction optionnelle d'un service fleet (réutilise les types `core`).
