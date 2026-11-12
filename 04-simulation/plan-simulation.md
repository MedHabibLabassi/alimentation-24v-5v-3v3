# Plan de simulation

Version 0.1,

## 0. Règles du jeu
- Chaque simulation a un **critère de réussite chiffré** et une **valeur
  calculée** à laquelle on compare le résultat. Tout écart supérieur à 20 %
  est expliqué.
- Une simulation avec des composants idéaux ne prouve rien : on utilise les
  modèles des fabricants quand ils existent, et on le note.
- Chaque test produit : le fichier de simulation (.asc ou .cir), une capture
  d'écran et une ligne dans le tableau de résultats (section 5).
- Ce qui n'est pas simulable (thermique réelle, CEM, vieillissement) est dit
  explicitement dans le rapport.

## 1. Outils et modèles
| Besoin | Outil | Remarque |
|---|---|---|
| Convertisseurs (démarrage, échelon, Bode, thermique) | WEBENCH Power Designer (TI) | Simulation électrique et thermique en ligne |
| Convertisseurs avec modèle fabricant | PSpice for TI ou TINA-TI | Vérifier sur la page produit si un modèle existe pour la référence choisie |
| Protections d'entrée (surge, inversion, branchement à chaud) | LTspice | Modèles SPICE du fabricant pour le MOSFET, la zener, la TVS |
| Pire cas | LTspice (.step, .param, mc() / gauss()) | |

Les modèles de circuits intégrés TI ne se chargent généralement pas dans
LTspice. Dans ce cas, on utilise WEBENCH ou PSpice for TI, et on le dit
dans le rapport. Les valeurs de référence des composants viennent de mes
hypothèses de l'étape 3 : les remplacer par les fiches techniques.

## 2. Liste des simulations

### S1. Démarrage
- Configuration : Vin = 18, 24, 32 V ; charge nulle puis pleine
  (V5 : 1 A, V3V3 : 0,5 A)
- Mesures : V5, V3V3, courant d'entrée, instant d'activation du buck 3,3 V
- Critères : montée monotone ; dépassement < 5 % ; V3V3 démarre après V5
  (ordre V24 → V5 → V3V3) ; pas de redémarrages en boucle
- Référence : soft-start du circuit (à lire dans la fiche)

### S2. Régulation statique et rendement
- Configuration : balayage Vin 18 à 32 V, charge 0 à 100 %
- Critères : V5 et V3V3 dans ±3 % ; rendement global comparé aux calculs
- Références calculées : rendement global 86,6 % (18 V), 85,9 % (24 V),
  85,0 % (32 V) ; buck 24→5 V : 88,6 / 87,7 / 86,7 % ; buck 5→3,3 V : 93,3 %

### S3. Échelon de charge
- Configuration : V5 de 0,1 à 0,9 A ; V3V3 de 0,05 à 0,45 A ; front de
  1 A/µs ; Vin = 18, 24, 32 V
- Mesures : écart maximal, temps de rétablissement
- Critère : écart dynamique < ±5 % (spécification)
- Référence calculée : V3V3 ≈ 64 mV (1,9 %), estimation approximative

### S4. Stabilité et interaction entre les deux bucks
- Configuration : diagramme de Bode de chaque boucle ; impédance de sortie
  du buck 5 V ; impédance d'entrée du buck 3,3 V
- Critères : marge de phase > 45° (viser 60°) ; marge de gain > 10 dB ;
  |Zout,5V| au moins 10 fois inférieure à |Zin,3V3| dans la bande
  d'intérêt (critère de Middlebrook)
- Référence calculée : |Zin,3V3| ≈ 14 Ω

### S5. Ondulation de sortie
- Configuration : pleine charge, Vin = 18, 24, 32 V, bande limitée à 20 MHz
- Critère : < 50 mV crête à crête
- Références calculées : V5 ≈ 6 mV, V3V3 ≈ 1,4 mV (sans les pointes
  de commutation dues aux inductances parasites : les ajouter au modèle
  quand le routage existe)

### S6. Surge de dimensionnement (TVS)
- Configuration : impulsion de courant 9 A, montée 10 µs, décroissance
  exponentielle (50 % à 1 ms, τ ≈ 1,44 ms), injectée au connecteur, carte
  alimentée à 32 V
- Mesures : tension maximale après la TVS, tension aux bornes de la diode
  Schottky et du circuit intégré, tension des condensateurs, courant dans
  le MOSFET et le fusible
- Critères : tension du nœud d'entrée < 53,3 V ; chaque composant sous sa
  tension maximale ; I²t du fusible < 20 % de l'I²t de fusion
- Références calculées : estimation linéaire de la TVS (VBR 36,7 V +
  1,47 Ω × I) : ≈ 49,9 V à 9 A ; limite de la fiche : 53,3 V à 11,3 A ;
  I²t du surge ≈ 0,06 A²s (9² × 1,44 ms / 2)
- Modèle de TVS de secours (comportemental, à tester) :
  `B1 n 0 I=if(V(n)>36.7,(V(n)-36.7)/1.47,0)`. Le remplacer dès que
  possible par le modèle SPICE du fabricant.
- Point à observer : si la tension à 9 A est de 49,9 V et non 53,3 V, la
  marge de la diode Schottky (60 V) passe de 11 % à environ 17 %.

### S7. Inversion de polarité
- Configuration : entrée à −32 V, puis −18 V
- Critères : courant dans la carte < 1 mA ; tension côté carte ≈ 0 V
  (aucune tension négative) ; Vds du MOSFET = 32 V (< 60 V) ; Vgs dans
  ±20 V
- Référence calculée : Vds = 53 % de la limite

### S8. Branchement à chaud
- Configuration : échelon de 0 à 32 V ; inductance du câble 1 µH
  (aussi 0,3 et 3 µH) ; résistance fusible 0,15 Ω ; MOSFET avec modèle
  complet incluant la diode interne ; ESR de l'électrolytique 0,05 / 0,3 / 1 Ω
- Mesures : pic de courant dans le MOSFET et le fusible, pic de tension,
  I²t
- Critères : pic de courant inférieur au courant impulsionnel de la diode
  interne de la fiche ; I²t < 20 % de celui du fusible ; tension < 53,3 V
- Références (intégration numérique simplifiée) : ≈ 48 à 74 A, tension
  < 41 V, I²t ≈ 0,03 à 0,08 A²s (voir `protections-entree.md`, section 5)

### S9. Pire cas
- Variations : Vref ±1,5 % (buck 5 V) et ±1 % (buck 3,3 V) ; résistances
  0,1 % ; inductances ±20 % ; céramiques −50 % de capacité ; ESR minimale
  et maximale ; Vin = 18 et 32 V
- Méthode : cas extrêmes d'abord (4 à 8 combinaisons), puis Monte Carlo
  (≥ 100 tirages) pour la tension de sortie
- Critères : tensions statiques dans ±3 % ; marge de phase > 45° dans tous
  les cas ; ondulation < 50 mV
- Références calculées : V5 entre 4,90 et 5,07 V ; V3V3 entre 3,258 et
  3,334 V

### S10. Court-circuit en sortie
- Configuration : court-circuit franc sur V5, puis sur V3V3
- Mesures : courant de crête dans l'inductance et en entrée, comportement
  (limitation, redémarrage en rafale), reprise
- Critères : courant d'inductance < Isat ; pas de dépassement du courant
  maximal du circuit ; reprise correcte après retrait du défaut

### S11. Défaut en court-circuit de la TVS (analyse sur papier)
- Courant fourni par une alimentation 24 V typique de l'équipement client
  contre la courbe temps-courant du fusible. Documenter le temps de
  coupure et l'absence d'incendie ou de dégradation du PCB.

## 3. Pistes de décision selon les résultats
| Constat | Action |
|---|---|
| Diode Schottky > 54 V pendant S6 | Circuit RC sur le nœud SW, ou diode 100 V avec nouvelle étude thermique |
| Courant de S8 > courant impulsionnel de la diode interne | MOSFET plus robuste, résistance de 1 Ω ou NTC, ou moins de capacité d'entrée |
| Marge de phase < 45° (S4) | Modifier la compensation ou le condensateur de sortie |
| Instabilité d'interaction (S4) | Filtre d'entrée du buck 3,3 V ou réduction de la capacité |
| V statique hors ±3 % en S9 | Resserrer les tolérances, ou assouplir la spécification (documenter) |

## 4. Organisation du dossier
04-simulation/
├── plan-simulation.md
├── webench/ (captures et exports)
├── ltspice/ (fichiers .asc, une simulation par fichier)
├── captures/ (nom : S3_echelon_V5_24V.png)
└── resultats.md

## 5. Tableau de résultats (à remplir)
| Test | Condition | Valeur calculée | Valeur simulée | Écart | Critère | Verdict |
|---|---|---|---|---|---|---|
| S1 | | | | | | |
| S2 | | | | | | |
| S3 | | | | | | |
| S4 | | | | | | |
| S5 | | | | | | |
| S6 | | | | | | |
| S7 | | | | | | |
| S8 | | | | | | |
| S9 | | | | | | |
| S10 | | | | | | |
