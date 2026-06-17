# Modèle de policy & réponse — Aegis

Comment Aegis décide quoi faire d'une menace, comment l'utilisateur arbitre, et
comment on évite que le poste devienne inutilisable à cause de faux positifs.

## Modes

Deux modes, réglables **globalement ou par catégorie de menace** :

- **Detection** : observe, corrèle, alerte, journalise. Ne bloque pas l'exécution.
  Peut mettre un fichier en quarantaine sur match signature (réversible).
- **Prevention** : bloque/tue dans le kernel sur seuil atteint. fanotify répond
  `DENY` aux accès bloquants ; eBPF envoie SIGKILL avant complétion du syscall.

Réglage par catégorie : on peut vouloir `Ransomware = Prevention` (toujours) tout
en gardant `ExecHeuristic = Detection` (le temps de bâtir la baseline). Défaut
livré : **Detection global**, sauf `Ransomware` et `MmapWX` en **Prevention**
(menaces trop rapides pour une validation humaine).

## Réponse graduée par sévérité

| Sévérité | Mode Detection | Mode Prevention |
|---|---|---|
| Info | log | log |
| Low | log + notif discrète | log + notif |
| Medium | notif + quarantaine si fichier | notif + quarantaine + blocage accès |
| High | notif forte + quarantaine + isolation suggérée | quarantaine + isolation process |
| Critical | alerte forte + recommandation kill | **kill in-kernel** + quarantaine |

`Isolate` = on gèle/limite le process (cgroup freezer, coupe ses sockets) sans le
tuer, pour investigation. `Kill` = SIGKILL immédiat.

## Corrélation (le multiplicateur de valeur)

Un signal isolé reste prudent ; la corrélation escalade. Exemple :
`exec depuis /tmp` (Medium) **+** `connexion sortante non résolue` (Medium) **+**
`capset` (High) → un seul incident **Critical** mappé sur une chaîne d'attaque.
C'est ce qui distingue un EDR d'un scanner : on raisonne sur des séquences, pas
des événements isolés. Fenêtre de corrélation par arbre de processus (ppid).

## Faux positifs — l'utilisateur garde le contrôle

- **Baseline** : à la première installation, période d'apprentissage qui recense
  les process/chemins/connexions habituels → réduit le bruit initial.
- **Exclusions** : par chemin, hash ou process. Explicites, listées, révocables.
- **Feedback boucle courte** : sur chaque alerte, l'UI propose « légitime » →
  génère une exclusion suggérée (jamais appliquée silencieusement).
- **Jamais d'auto-exclusion** : une menace ignorée est toujours une décision
  utilisateur tracée, pas une heuristique opaque.

## Déploiement & permissions

- **Daemon** : service systemd, démarré au boot. Privilèges minimaux nécessaires —
  `CAP_BPF`, `CAP_SYS_ADMIN` (fanotify), `CAP_DAC_READ_SEARCH` (scan), `CAP_KILL`.
  Pas de root nu si les capabilities suffisent.
- **Socket Unix** : permissions restreintes à un groupe `aegis` ; l'UI doit y
  appartenir. Aucun port réseau.
- **UI** : zéro privilège. Ne peut que demander ; le daemon arbitre et applique.
- **Auto-protection** : toute tentative d'un tiers d'arrêter/altérer le daemon ou
  ses sondes est elle-même une détection Critical (cf. T1562.001).

## Journalisation

`tracing` (équivalent Rust de pino) : tout verdict ≥ High et toute commande
mutante sont journalisés avec l'`event_id`, horodatés, dans `logs/` (reset au
restart pour le runtime, persistance séparée pour l'historique d'incidents).
