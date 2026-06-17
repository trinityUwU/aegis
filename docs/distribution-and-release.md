# Distribution & releases — Aegis

Un AV doit s'installer comme un logiciel système de confiance : par le
gestionnaire de paquets, signé, vérifiable, sans installeur opaque.

## Cibles de packaging

| Format | Cible | Priorité |
|---|---|---|
| **AUR** (`aegis`, `aegis-bin`) | Arch / dérivés (poste de Chris en premier) | v1.0 — prioritaire |
| **.deb** | Debian / Ubuntu / Mint | v1.0 |
| **.rpm** | Fedora / RHEL / openSUSE | v1.0 |
| **Flatpak** | universel (sandbox — limites pour un AV à documenter) | v1.x |
| **AppImage** | portable sans install | v1.x |
| Binaire statique **musl** | self-contained, distros exotiques | v1.0 |

Note Flatpak : le sandbox Flatpak restreint l'accès kernel (eBPF/fanotify) —
le packaging Flatpak nécessitera des permissions élevées explicites, voire ne
couvrira que l'UI cliente avec un daemon installé hors Flatpak. À trancher en v1.x.

## Composants installés

- Binaire `aegis-daemon` + `aegisctl`.
- **Unit systemd** (`aegis.service`) : démarrage au boot, restart on-failure,
  durcissement (`ProtectSystem`, `NoNewPrivileges`, capabilities ciblées).
- Groupe système `aegis` (accès socket pour l'UI).
- Règles embarquées dans `/usr/share/aegis/rules/`, surcharge utilisateur dans
  `/var/lib/aegis/rules/`.
- **Capabilities minimales** plutôt que root nu : `CAP_BPF`, `CAP_SYS_ADMIN`
  (fanotify), `CAP_DAC_READ_SEARCH`, `CAP_KILL`.
- Script post-install : détection de la config kernel (BPF-LSM), **message clair**
  si une activation GRUB + reboot est nécessaire — proposé, jamais imposé.

## Découplage code / règles
Le code suit le cycle de releases (paquets). Les **règles et IOC se mettent à jour
indépendamment** (cf. `threat-intel.md`) : on n'attend pas une release pour
pousser une nouvelle signature.

## Canaux de release

| Canal | Contenu | Public |
|---|---|---|
| `nightly` | build auto sur `master`, non signé garanti stable | dev |
| `beta` | préversions taguées, testables | early adopters |
| `stable` | releases SemVer **signées** | tout public |

## Signature & intégrité des releases
- Artefacts signés (GPG et/ou **sigstore/cosign**) ; checksums publiés.
- Objectif **reproducible builds** (un AV doit être vérifiable bit à bit) — visé
  dès que la toolchain le permet.
- `CHANGELOG.md` tenu par version (Keep a Changelog), notes de release lisibles
  (passées au `/humanizer`).

## Versioning
- **SemVer** strict. `MAJOR` = rupture (format de règles, IPC) ; `MINOR` =
  features ; `PATCH` = corrections.
- Le contrat IPC (`aegis-core`) porte son propre `schema_version` indépendant.
