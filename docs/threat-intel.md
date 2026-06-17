# Threat intelligence — Aegis

Comment Aegis sait ce qui est malveillant, et comment il l'apprend sans jamais
trahir le principe local-first.

## Principe directeur

Les mises à jour de renseignement sont **tirées (pull), jamais poussées**, et
**jamais imposées** : déclenchées par l'utilisateur ou planifiées par lui, ou
livrées sous forme de bundle hors-ligne. Aucun phone-home. Aucune donnée
utilisateur ne sort. Le renseignement entre, rien ne sort.

## Sources open source agrégées

| Source | Type | Usage |
|---|---|---|
| abuse.ch — **MalwareBazaar** | échantillons + hash | jeux de hash malveillants, corpus de test |
| abuse.ch — **ThreatFox** | IOC (hash, IP, domaines) | store d'IOC, heuristiques réseau |
| abuse.ch — **URLhaus** | URLs malveillantes | protection téléchargements (host-side) |
| abuse.ch — **YARAify** | règles YARA | bundle de règles |
| **Neo23x0 / signature-base** (Florian Roth) | règles YARA | détection APT/malware éprouvée |
| **Elastic protections-artifacts** | règles YARA + comportementales | détection moderne, licence permissive |
| **Yara-Rules** (communauté) | règles YARA | couverture large |

Sélection au cas par cas selon compatibilité de **licence** (priorité aux
licences permissives, compatibles MIT/Echo) — tracé dans `rules/SOURCES.md`.

## Format & organisation des règles

- Règles YARA sous `rules/yara/<famille>/*.yar`, compilées par `yara-x`.
- Règles comportementales (corrélation, seuils) sous `rules/behavior/*.toml`.
- IOC sous `rules/ioc/` (hash, domaines, IP) chargés par `aegis-intel`.
- ~1 % des règles YARA importées nécessitent une revue manuelle (incompatibilités
  yara-x) — étape de validation au pipeline d'agrégation.

## Pipeline de mise à jour

1. **Récupération** : pull des sources activées (HTTPS, déclenché/planifié).
2. **Vérification** : signature du bundle (GPG ou sigstore) + checksums. Un
   bundle non vérifié est rejeté — surface critique (cf. `security-and-governance.md`).
3. **Validation** : compilation `yara-x` à blanc, rejet des règles cassées,
   détection des règles trop bruyantes (faux positifs connus).
4. **Activation atomique** : swap du jeu de règles sans interrompre la protection.
5. **Versioning** : chaque jeu de règles a une version + date, rollback possible.

## Bundle hors-ligne

Pour les postes sans réseau (souveraineté totale) : un bundle signé téléchargeable
ailleurs et importé manuellement (`aegisctl intel import <bundle>`). Garantit la
mise à jour sans aucune connexion sortante.

## Réputation de hash locale

Base locale de hash malveillants connus (issue de MalwareBazaar) interrogée avant
un scan complet : un hash connu = verdict immédiat sans coût de scan. Jamais
d'interrogation cloud — la base est locale et mise à jour comme les règles.

## Cadence
- Règles YARA / IOC : mise à jour quotidienne planifiable (défaut), ou manuelle.
- Code applicatif : via le gestionnaire de paquets de la distro (cf.
  `distribution-and-release.md`), découplé des règles.
