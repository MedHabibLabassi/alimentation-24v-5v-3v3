# Calculs : protections d'entrée et déclassement

## 0. Hypothèses (à remplacer par les fiches techniques)
| Paramètre | Valeur supposée | Source à vérifier |
|---|---|---|
| Fusible | 1 A, fusion lente, CMS, 63 V DC | Fiche technique |
| Résistance à froid du fusible | 0,15 Ω | Fiche technique |
| Facteur de température du fusible à 60 °C | 0,9 | Courbe du fabricant |
| I²t de fusion du fusible | ≥ 0,5 A²s | Fiche technique |
| P-MOSFET | canal P, 60 V, Rds(on) 40 mΩ à Vgs = −10 V | Fiche technique |
| Vgs max du MOSFET | ±20 V | Fiche technique |
| Zener de grille | 12 V ±5 % (11,4 à 12,7 V), 300 mW | Fiche technique |
| Résistance de grille | 47 kΩ, 0603, 75 V max | Fiche technique |
| TVS | SMBJ33A : VRWM 33 V, VBR min 36,7 V, Vc 53,3 V à 11,3 A, 600 W (10/1000 µs) | Fiche technique |
| Inductance du câble d'alimentation | 1 µH | Estimation |
| Capacité d'entrée totale | 2 × 4,7 µF + 47 µF ≈ 52 µF | Parties A et B1 |

## 1. Courants et pertes de la chaîne d'entrée
Puissance au point 5 V (1 A + buck 3,3 V) : 6,77 W. Rendement du buck 24→5 V
repris de la partie A.
| Vin | Pin buck | Courant d'entrée | Perte fusible | Perte MOSFET |
|---|---|---|---|---|
| 18 V | 7,64 W | 0,43 A | 27 mW | 7 mW (11 mW à chaud) |
| 24 V | 7,72 W | 0,32 A | 16 mW | 4 mW |
| 32 V | 7,81 W | 0,24 A | 9 mW | 2 mW |

Rendement global (Pout = 6,65 W, protections comprises) :
| Vin | Pin total | Rendement |
|---|---|---|
| 18 V | 7,68 W | 86,6 % |
| 24 V | 7,74 W | 85,9 % |
| 32 V | 7,82 W | 85,0 % |

Les protections coûtent moins de 0,05 W (0,6 %). Courant de dimensionnement
conservé : 0,5 A.

## 2. Fusible
Calibre minimal : In ≥ Imax / (0,75 × kT) = 0,5 / (0,75 × 0,9) ≈ 0,74 A
→ calibre normalisé retenu : **1 A, fusion lente**.
- Courant en service à 18 V : 0,43 A, soit 43 % du calibre
- Tension : 63 V DC, soit 51 % de la tension d'entrée maximale en marge
- Pouvoir de coupure : doit dépasser le courant de court-circuit de
  l'alimentation du client (à vérifier, valeur typique des CMS : 35 A ou plus)
- I²t d'un surge 10/1000 µs de 9 A : ≈ 81 × 0,5 ms ≈ 0,04 A²s
  (< 10 % de l'I²t de fusion supposé)

PTC écarté : son courant de maintien baisse avec la température
(à 60 °C ambiants) et, si la TVS se met en court-circuit, un PTC
laisse la TVS chauffer. Le fusible coupe franchement.

Rôle du fusible : il ne protège pas contre le court-circuit de sortie (le
buck gère le sien par limitation de courant). Il isole la carte quand un
composant d'entrée tombe en court-circuit (TVS, MOSFET, condensateur).

## 3. P-MOSFET anti-inversion
Montage : MOSFET en série sur le + (drain côté connecteur, source côté carte),
grille reliée à la masse par une résistance de 47 kΩ, zener de 12 V entre la
grille (anode) et la source (cathode).

Fonctionnement :
- Polarité correcte : la source monte à Vin (d'abord par la diode interne),
  la grille reste à 0 V, donc Vsg = Vin limité par la zener à 12 V : MOSFET passant
- Polarité inversée : la diode interne est bloquée et Vgs ≈ 0 : MOSFET bloqué.
  La carte (condensateurs polarisés compris) ne voit aucune tension inverse

Tensions de grille :
| Vin | Vsg | Rapport à Vgs max (20 V) |
|---|---|---|
| 18 V | 10,7 à 12,7 V (zener peu polarisée : tension un peu plus basse) | 64 % |
| 32 V | 11,4 à 12,7 V | 64 % |

Le Vsg minimal (10,7 V) reste supérieur au Vgs de 10 V auquel Rds(on) est spécifié.

Courants et pertes de la polarisation :
| Cas | Courant (Vin − Vz) / R | Puissance zener | Puissance résistance |
|---|---|---|---|
| 18 V | 0,13 mA | 1,6 mW | 1 mW |
| 32 V | 0,43 mA | 5,2 mW | 8,5 mW |
| Surge (53,3 V) | 0,86 mA | 10,9 mW | 35 mW |

Choix de Vds : 60 V. En inversion à 32 V, Vds = 32 V, soit 53 % (80 % avec un
MOSFET 40 V : insuffisant). Aucun condensateur n'est placé en amont du
MOSFET, donc pas de résonance avec le câble en inversion.

Échauffement : P = I² × Rds(on) ≈ 0,43² × 0,06 (à chaud) ≈ 11 mW,
θJA = 100 °C/W → +1 °C. Même avec Rds(on) = 200 mΩ, la perte reste sous 50 mW.

## 4. TVS (SMBJ33A)
| Critère | Condition | Valeur | Verdict |
|---|---|---|---|
| Tension de non-conduction | VRWM ≥ Vin max (32 V) | 33 V (marge 1 V) | OK, marge faible |
| Début de conduction | VBR min > Vin max | 36,7 V | OK |
| Tension d'écrêtage | Vc < limite du buck (60 V) | 53,3 V (marge 6,7 V) | OK, à surveiller |
| Surge de dimensionnement | ≤ 80 % de Ipp (11,3 A) | 9 A, 10/1000 µs | OK |
| Polarité | Unidirectionnelle | MOSFET en amont bloque l'inversion | OK |

Le Vc de 53,3 V est la valeur au courant maximal de la fiche. À 9 A il est plus
faible : valeur à lire sur la courbe Vc/Ipp du fabricant. On garde 53,3 V
partout, ce qui est prudent.

Limite connue : la TVS protège contre les transitoires, pas contre une
surtension permanente. Si l'entrée reste entre 36,7 V et 60 V, la TVS conduit
et chauffe jusqu'à la défaillance (en court-circuit en général) puis le fusible
coupe. La carte est sauvée, mais la TVS et le fusible sont sacrifiés. Une
protection active (comparateur et MOSFET de coupure) serait nécessaire pour
tenir une surtension permanente : amélioration possible pour la V2.

## 5. Appel de courant à la mise sous tension
Capacité totale vue par le câble : C ≈ 52 µF. Énergie : ½ × C × V² = 27 mJ à 32 V.
Deux bornes pour le pic de courant :
- Limité par le câble (L = 1 µH) : Z = √(L / C) ≈ 4,4 Ω → pic ≈ 32 / 4,4 ≈ 7 A,
  durée ≈ π × √(L × C) ≈ 23 µs, I²t ≈ 5 × 10⁻⁴ A²s
- Limité par la seule résistance série (R = 0,5 Ω, cas pessimiste) :
  pic 64 A pendant ≈ 26 µs, I²t = V² × C / (2R) ≈ 0,05 A²s

Conclusions :
- I²t pessimiste = 0,05 A²s, soit 10 % de l'I²t de fusion supposé (0,5 A²s) : acceptable
- Un branchement à chaud sans amortissement peut faire monter la tension
  jusqu'à 2 × Vin. Le condensateur électrolytique de 47 µF (ESR élevée)
  amortit ce phénomène, et la TVS écrête le reste
- Le pic de courant traverse la diode interne du MOSFET : à comparer avec
  son courant impulsionnel (Idm) dans la fiche. Si le pic dépasse Idm,
  ajouter une résistance de 1 Ω en série ou choisir un MOSFET plus robuste

## 6. Condensateurs d'entrée : changement de tension nominale
Avec Vc = 53,3 V, les céramiques 50 V de la partie A sont dépassées. Nouveau choix :
- 2 × 4,7 µF, **100 V**, X7R ou X7S, boîtier 1210
- 1 × 47 µF, **63 V**, électrolytique ou polymère (ESR amortissante)

Perte de capacité : le calcul de la partie A utilise 50 % à 24 V. Une céramique
100 V perd moins (≈ 30 % selon les courbes habituelles). L'hypothèse de 50 %
reste donc prudente. À vérifier sur la courbe DC-bias du fabricant.

## 7. Tableau de déclassement complet
Valeurs normales prises au pire cas de service. « Surge » : transitoire à 53,3 V.
| Composant | Paramètre | Limite | Normal | Taux | Surge | Taux | Statut |
|---|---|---|---|---|---|---|---|
| Buck 24→5 V | Tension d'entrée | 60 V | 32 V | 53 % | 53,3 V | 89 % | OK, surge à surveiller |
| Buck 24→5 V | Courant de sortie | 3,5 A | 1,4 A | 40 % | | | OK |
| Diode Schottky | Tension inverse | 60 V | 32 V | 53 % | 53,3 V | 89 % | Surge à surveiller |
| Diode Schottky | Courant moyen | 3 A | 1,18 A | 39 % | | | OK |
| Diode Schottky | Tj (limite déclassée) | 100 °C | ≈ 90 °C | | | | OK, 10 °C de marge |
| Inductance 22 µH | Irms | ≥ 1,8 A | 1,40 A | 78 % | | | OK |
| Inductance 22 µH | Isat | ≥ limite de courant du circuit | 1,59 A | à calculer | | | À fixer dans la fiche |
| Cin céramique | Tension | 100 V | 32 V | 32 % | 53,3 V | 53 % | OK |
| Cin électrolytique | Tension | 63 V | 32 V | 51 % | 53,3 V | 85 % | OK, surge à surveiller |
| Cout 5 V | Tension | 16 V | 5,15 V | 32 % | | | OK |
| P-MOSFET | Vds | 60 V | 32 V | 53 % | | | OK |
| P-MOSFET | Vgs | 20 V | 12,7 V | 64 % | 12,7 V | 64 % | OK |
| P-MOSFET | Courant continu | ≥ 2 A | 0,43 A | 22 % | | | OK |
| Zener de grille | Puissance | 300 mW | 5,2 mW | 2 % | 10,9 mW | 4 % | OK |
| Résistance de grille | Tension | 75 V | 20 V | 27 % | 41 V | 55 % | OK |
| Résistance de grille | Puissance | 100 mW | 8,5 mW | 9 % | 35 mW | 35 % | OK |
| TVS | VRWM / Vin max | 33 V | 32 V | 97 % | | | Marge faible, par construction |
| TVS | Puissance de surge | 600 W | | | 480 W | 80 % | OK |
| Fusible | Courant en service | 1 A | 0,43 A | 43 % | | | OK |
| Fusible | I²t d'appel | 0,5 A²s | 0,05 A²s | 10 % | | | OK |
| Buck 5→3,3 V | Voir partie B1 | | | | | | OK |

Seuil de référence : 70 à 80 % en régime permanent. Les taux du surge sont des
transitoires rares et brefs. Le statut « surge à surveiller » signale les
marges inférieures à 15 % pendant ces événements.

## 8. Constats et points ouverts
1. Diode Schottky : marge de 11 % pendant le surge. Options : (a) garder
   60 V et réduire les oscillations du nœud SW par un routage soigné
   (éventuellement un circuit RC de dissipation, à simuler) ; (b) passer à
   100 V, avec une Vf plus élevée (≈ 0,65 V) qui porte la perte à ≈ 0,77 W
   et Tj à ≈ 98 °C, donc sans marge thermique. Option (a) retenue, à valider à
   l'étape 4
2. Inductance : fixer Isat dans la fiche du circuit (limite de courant) et
   recalculer la perte DCR avec la valeur réelle (un modèle 5 A a souvent un
   DCR plus faible que 0,1 Ω)
3. Surtension permanente (> 36,7 V) non couverte : limite connue
4. MOSFET : comparer le pic d'appel de courant à Idm
5. Remplacer toutes les hypothèses par les valeurs des fiches techniques
