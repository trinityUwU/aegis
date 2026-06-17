# Politique de sécurité — Aegis

Aegis est un logiciel de sécurité tournant en privilégié : nous traitons nos
propres vulnérabilités avec la rigueur que nous appliquons aux menaces.

## Signaler une vulnérabilité

Merci de **ne pas** ouvrir d'issue publique pour une faille de sécurité.

Contactez le mainteneur en privé (divulgation responsable). Indiquez :
- une description de la vulnérabilité et de son impact,
- les étapes de reproduction,
- la version concernée.

Un délai raisonnable de correction est observé avant toute divulgation publique.

## Périmètre

Sont particulièrement critiques : le daemon privilégié, les sondes eBPF/fanotify,
le pipeline de mise à jour des règles, le socket de contrôle local, et le parsing
de fichiers potentiellement hostiles. Détail du modèle de menace :
[`docs/security-and-governance.md`](./docs/security-and-governance.md).

## Limites assumées

Aucun EDR n'attrape 100 % des menaces. Aegis ne prétend pas couvrir les zero-days
purement inédits ni les rootkits kernel qui subvertissent eBPF/LSM eux-mêmes. Ces
limites sont documentées, pas masquées.
