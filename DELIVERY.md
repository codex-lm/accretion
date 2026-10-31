# Note de livraison — Accretion

Z Code, 31 octobre 2026. Prototype livré pour essai par Cédric.
Publication LOB7 volontairement **différée** jusqu'à son retour.

- Dépôt : https://github.com/codex-lm/accretion
- Jouable : ouvrir `index.html` (fichier unique autonome, ~90 Kio, aucune dépendance).

## Ce qui a été réellement testé

**Tests automatisés (Node, 18/18 verts)** — logique extraite directement de
`index.html` (source unique, marqueurs `__CORE__`) :

- placement sur la couche libre la plus basse, trous flottants ;
- détection d'anneau complet indépendante des trous ;
- explosion → retrait de l'anneau → **compactage gravitaire par colonne** ;
- cascades ×2 (trou refermé) et ×3 (escalier de trous), scores exacts
  (100 × couche × position de chaîne, pur ×3 — cas vérifié à 1 500 points) ;
- comète (détruit le sommet, secteur vide = rien), noyau instable (détruit
  les voisins de SA couche, refus si colonne pleine), barre (2 secteurs
  adjacents structurellement, jamais la même colonne sur une frontière) ;
- courbe de difficulté (intervalle 2,2 s → 0,6 s, salves 1→3, couleurs 3→5,
  vitesse ×1→×2, niveaux tous les 8 anneaux) ;
- i18n EN/FR/ES complète ; fuzz 50 000 opérations légales sans violation
  d'invariant.

**Simulation de parties (300+ parties, bot à rotation partagée)** : le bot
ne peut bien viser qu'un débris par salve, comme un joueur. Résultats :
novice médiane **4,8 min**, expert **6,7 min**, cascades ×2+ dans **66 %**
des parties de plus de 5 min pour un joueur qui ménage les trous (stratégie
récompensée : le bot « escalier » fait significativement mieux que le naïf).

**Navigateur réel ( Chromium intégré, session documentée)** :

- écran d'accueil, tutoriel (11/12 secteurs préremplis, 1er anneau fermé
  en < 20 s), marqueurs d'atterrissage recalculés en continu ;
- atterrissages visés via le marqueur : secteur exact respecté ;
- boost (+5), comète, noyau instable (destruction + bannière d'astuce),
  barre — chacun vérifié sur l'état de la grille ;
- cascade ×2 pure déclenchée en conditions réelles : 1 500 points,
  ralenti, chaîne affichée ;
- défaite par dépassement de la 8ᵉ couche, écran de fin (score, record,
  chaîne), **Rejouer mesuré à 0 ms** ;
- pause (P/Échap), pause automatique sur perte de focus, bascules son et
  mouvements réduits, cycle de langues FR→ES→EN vérifié (textes + `lang`) ;
- mode `?debug=1` (FPS · intervalle · niveau) ;
- vue portrait 390×844 : aucun chevauchement, boutons ≥ 38 px ;
- captures dans `../evidence/accretion/` (3 PNG).

**Non testé** : manette (non implémentée — facultative dans la spec),
téléphone physique (test tactile simulé uniquement), audio qualitatif
entendu en synthèse mais pas validé en conditions réelles de salle.

## Réglages choisis (CFGC en tête de script)

- Difficulté pilotée en continu par les anneaux nettoyés : intervalle
  2,2 × 0,91^anneaux (borné 0,6 s), salves simultanées 1/2/3 aux anneaux
  6/14, couleurs 3/4/5 aux anneaux 14/24, vitesse ×1→×2 vers l'anneau 28.
  La spec lie les intervalles à « vers 6 minutes » : ce calage les atteint
  vers 5-6 min en jeu réel (simulé), les TYPES restent par niveau (barre
  niv. 2, comète niv. 3, noyau instable niv. 4, soit 8/16/24 anneaux).
- Noyau instable 10 % (« rare » : ~1 débris sur 10 au niveau 4+), comète
  8 %, barre 18 %.
- Rotation : accélération 55 rad/s², plafond 9 rad/s (demi-tour ≈ 0,43 s),
  aimant léger vers l'alignement des secteurs au relâcher.

## Décisions d'interprétation de la spec

1. **Cascades** : « les blocs retombent d'un cran… on revérifie » — pour que
   des cascades ×2+ soient possibles (critère d'acceptation), la chute
   compacte chaque colonne (gravité), et les trous créés par les noyaux
   instables persistent jusqu'à une explosion. Sans cela, la chaîne est
   mathématiquement toujours ×1. L'astuce enseignée en jeu : garder les
   trous pour les reboucher en dernier → tout l'escalier s'effondre.
2. **« Débris simultanés en approche 1 → 3 »** : implémentés en salves
   simultanées (1, puis 2, puis 3 d'un coup) — c'est ce qui crée le dilemme
   de la rotation globale (« tu choisis lequel bien placer »).
3. **Défaite des barres** : si l'une des deux moitiés dépasse la 8ᵉ couche,
   la partie s'arrête (le débris est indivisible).

## Limites connues

- Les symboles daltoniens des blocs sont petits en portrait (< 400 px) ;
  les couleurs restent le signal principal.
- Le marqueur d'une comète vise le sommet actuel : si la colonne change
  pendant l'approche, il suit (recalculé à chaque image) — visuellement
  exact, mais vif.
- L'audio dépend des politiques de lecture du navigateur (démarrage au
  premier geste, coupé sinon) — comportement voulu.
- Environnement de test intégré bridé (~4-20 i/s affichés) : le garde-fou
  anti-gel de boucle (minuteur de secours) a été ajouté et vérifié ; les
  60 i/s d'un navigateur normal restent la cible (53 FPS observés en phase
  active).

## Graphisme et son

Famille visuelle « espace graphite sobre » (consigne de variété du 31/10) :
fond graphite neutre #0c0e11 — délibérément hors du gabarit nuit
indigo/violet du site LOB7 —, noyau crème #ded4c2, cinq teintes vives
distinctes (corail, ambre, vert, bleu, rose) chacune avec son motif, rouge
danger unique. Tous les visuels et sons sont générés par le code, aucun
asset tiers. Couverture 100 % procédurale (outil Node dans le dépôt de dev,
hors dépôt public).
