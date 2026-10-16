# Project-Warzone — prototype « réaction de l'IA ennemie »

Prototype tactique (BRM5 / Operator-like) en **un seul fichier** : `index.html`.
Ouvre-le par double-clic (protocole `file://`) : aucune dépendance, aucun CDN,
aucun build. Tout est inline (HTML + CSS + JS ES2020, un seul `<script>`).

## Ce que valide ce prototype

L'IA ennemie : des soldats qui perçoivent, mémorisent, communiquent, se
coordonnent, et dont le comportement dépend de leur entraînement
(milice / infanterie régulière / forces spéciales). Chaque action du joueur
(bruit, cadavre, tir, disparition) a une conséquence visible sur la carte.

**Règle d'or** : un soldat ne lit JAMAIS la vraie position du joueur. Ses
décisions ne prennent en entrée que sa mémoire de contacts (vue, bruit,
soupçon, radio), ses ordres, sa doctrine, son moral et la géométrie statique.
Deux garde-fous l'imposent : le `ctx` du cerveau ne contient aucune référence
au joueur (et un Proxy lève une erreur si le code tente `ctx.player`), et les
self-tests piègent `G.player` pendant l'exécution du cerveau.

## Commandes

| Touche | Action |
|---|---|
| `ZQSD` / `WASD` / flèches | déplacement |
| souris | visée |
| clic gauche | tir (hitscan, bloqué par les murs) |
| `Shift` | sprint (bruyant, très visible) |
| `C` | accroupi (silencieux, discret, lent) — bascule |
| `Q` | lancer une pierre (bruit à l'endroit visé, 25 m) |
| `F1` | bascule vue debug (activée par défaut) |
| `R` | reset du scénario |
| `Espace` | pause |
| `+` / `-` | échelle de temps 0,25× → 4× |

Repères clavier : les déplacements utilisent le **code physique** des touches
(`KeyW/KeyA/KeyS/KeyD`), donc ZQSD et WASD fonctionnent sans réglage.
La pierre est sur la touche physique `Q` (soit `A` sur un clavier AZERTY).

## Paramètres d'URL

- `?seed=12345` — graine du générateur aléatoire (scénarios reproductibles).
- `?test=1` — exécute au chargement les 21 self-tests logiques **et** les
  9 scénarios d'acceptation, et affiche le compte rendu en plein écran
  (également dans la console).

## Vue debug (F1)

Murs, points de couverture, cônes de vision, état de chaque soldat en français
(`PATROUILLE`, `SUSPICIEUX`, `ENQUÊTE`, `ENGAGEMENT`, `COUVERT`, `SUPPRESSION`,
`FLANQUEMENT`, `RECHERCHE`, `REPLI`, `PANIQUE`, `CORPS`), jauges de moral et de
vie, mémoire des contacts (cercle = incertitude, opacité = confiance), chemins
et routes de débordement en pointillés, messages radio, bruits en anneaux,
comparaison « position vraie du joueur / meilleure croyance ennemie », bandeau
d'alerte et journal des événements (12 dernières lignes).
Vue debug coupée : seules les entités en ligne de vue du joueur sont dessinées.

## Architecture du fichier (`index.html`)

1. `CONFIG` — 2. `UTIL` — 3. `PERCEPTION` (`perception.js` inliné) —
4. `WORLD` (LOS, occlusion, grille, A*, couverture) — 5. `DOCTRINES` —
6. `RADIO` — 7. `SOLDAT + CERVEAU` (HSM) — 8. `SQUAD` — 9. `COMMAND` —
10. `JOUEUR & COMBAT` — 11. `RENDU & DEBUG UI` —
12. `SCÉNARIO & SELF-TESTS` — 13. `BOUCLE PRINCIPALE`.

Espace de noms global unique : `G`.
