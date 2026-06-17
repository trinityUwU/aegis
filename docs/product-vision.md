# Vision produit — Aegis

## Mission

La protection endpoint de référence sur Linux desktop : un EDR host complet,
temps réel, **100 % local, souverain et open source**, qui égale les capacités
des solutions propriétaires (CrowdStrike, SentinelOne) sans leur télémétrie cloud,
et qui reste utilisable par un humain — pas seulement par un SOC.

## Positionnement

Le marché Linux est fracturé en trois camps, aucun ne couvre le besoin :

| Catégorie | Exemples | Manque |
|---|---|---|
| Scanners de signatures | ClamAV | aucun comportemental, pas de temps réel natif |
| Runtime security cloud | Falco, Tetragon, Tracee | pensés Kubernetes/SRE, YAML/CLI, inutilisables sur desktop |
| EDR propriétaires | CrowdStrike, SentinelOne | fermés, cloud, télémétrie imposée, payants, non souverains |

**Aegis occupe le trou** : signatures **+** comportemental **+** réseau host **+**
intégrité **+** réponse **+** forensics, dans un produit unique, local, beau,
auditable, gratuit.

## Principes fondateurs (invariants produit)

1. **Local-first absolu** — aucune dépendance cloud, aucun phone-home imposé.
   Les mises à jour de règles sont déclenchées/planifiées par l'utilisateur ou
   livrées hors-ligne.
2. **Zéro télémétrie par défaut** — Aegis ne renvoie rien. Différenciateur réel
   face aux EDR US et argument RGPD natif.
3. **Perf-first** — budget CPU strict, filtrage in-kernel, **zéro GPU/VRAM**,
   aucun ML lourd. La protection ne doit jamais se sentir.
4. **Transparence** — open source = auditable. Un AV qu'on ne peut pas auditer
   est un acte de foi ; le nôtre est lisible.
5. **L'UX comme arme** — la sécurité runtime n'a jamais eu d'interface belle et
   compréhensible en open source. C'est notre angle d'attaque.
6. **Défense en profondeur** — chaque vecteur couvert par plusieurs couches
   indépendantes ; aucune couche n'est un point unique de défaillance.

## Ce qu'Aegis EST

Protection **host endpoint** complète : détection (signature + comportement +
réseau host + intégrité), prévention (blocage/kill in-kernel), réponse
(quarantaine, isolation, rollback), investigation (forensics, timeline, threat
hunting local).

## Ce qu'Aegis N'EST PAS (scope guard)

Volontairement hors périmètre — pour ne pas diluer le produit :
- **Pas une suite « sécurité grand public »** : ni VPN, ni gestionnaire de mots de
  passe, ni contrôle parental, ni webcam guard. Ce sont des produits distincts.
- **Pas un IDS/IPS réseau périmétrique** (NIDS type Suricata) : on surveille le
  réseau **du point de vue du host et des process**, pas le trafic réseau global.
- **Pas un SIEM cloud / XDR multi-domaines** : un module fleet local viendra plus
  tard, mais on ne devient pas une plateforme cloud.
- **Pas un scanner de vulnérabilités de paquets** (type Lynis/OpenVAS) : module
  possible à terme, pas le cœur.

## Horizon

- **An 1** : EDR desktop Linux complet et stable (jusqu'à v1.0), utilisable au
  quotidien par un tiers, packagé multi-distro.
- **An 2** : profondeur — rootkit/intégrité, sandbox de détonation, threat
  hunting, device control, rollback ransomware (v1.x → v2.0).
- **An 3** : extension contrôlée — fleet multi-host local optionnel, API externe
  opt-in, exploration d'une couche heuristique avancée CPU-only (jamais au prix
  des invariants perf). (v3.0)

Détail par version : [`roadmap-versions.md`](./roadmap-versions.md). Inventaire
exhaustif : [`feature-matrix.md`](./feature-matrix.md).
