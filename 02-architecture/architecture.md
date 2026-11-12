# Architecture et choix de conception

Version 0.2

Les rendements et résistances thermiques sont des hypothèses de départ,
à remplacer par les valeurs des fiches techniques. Le détail des calculs
est dans `03-calculs/`.

## 1. Bilan de puissance
| Élément | Calcul | Résultat |
|---|---|---|
| Puissance utile V5 | 5 V × 1 A | 5 W |
| Puissance utile V3V3 | 3,3 V × 0,5 A | 1,65 W |
| Puissance utile totale | | 6,65 W |
| Puissance d'entrée du buck 3,3 V | 1,65 W / 0,933 | ≈ 1,77 W |
| Puissance au nœud 5 V | 5 W + 1,77 W | ≈ 6,77 W |
| Courant demandé au buck 5 V | 1 A + 0,35 A | ≈ 1,35 A (calcul : 1,4 A) |
| Puissance d'entrée à 18 V | | ≈ 7,68 W |
| Courant d'entrée à 18 V | 7,68 W / 18 V | ≈ 0,43 A (dimensionnement : 0,5 A) |

Rendements (calculs de l'étape 3) :
- buck 24→5 V : 86,7 à 88,6 % selon la tension d'entrée
- buck 5→3,3 V : ≈ 93 %
- global, protections comprises : 85,0 à 86,6 %

## 2. Protection contre l'inversion de polarité

| Critère | Diode Schottky en série | P-MOSFET en série |
|---|---|---|
| Chute / résistance | Vf ≈ 0,5 V à 0,43 A | Rds(on) ≈ 40 mΩ (≈ 60 mΩ à chaud) |
| Perte à 0,43 A | ≈ 0,22 W | ≈ 11 mW (I²R) |
| Impact sur le rendement | ≈ 3 % | négligeable |
| Nombre de composants | 1 | 3 (MOSFET, zener, résistance) |
| Points d'attention | Tension inverse ≥ 60 V (32 V en inversion, marge nécessaire) | Vds ≥ 60 V ; Vgs max ±20 V : une zener de 12 V limite Vgs, car l'entrée monte à 32 V |
| Coût | Très faible | Faible |

**Choix : P-MOSFET canal P, 60 V**, avec zener de grille 12 V et résistance
de grille 47 kΩ. La perte est quasi nulle. La diode Schottky resterait une
option acceptable pour ce courant, mais le MOSFET demande plus de soin de
conception, ce qui est précisément ce que ce projet veut démontrer.

Vérifications effectuées (voir `03-calculs/protections-entree.md`) :
- Vds en inversion à 32 V : 53 % de la limite de 60 V (80 % avec un 40 V :
  insuffisant, d'où le choix du 60 V)
- Vgs maximal : 12,7 V, soit 64 % de la limite de 20 V
- Vsg minimal (10,7 V) supérieur au Vgs de 10 V auquel Rds(on) est spécifié

## 3. Génération du 3,3 V

| Critère | A : LDO 5→3,3 V | B : buck 5→3,3 V (en cascade) | C : buck 24→3,3 V (parallèle) |
|---|---|---|---|
| Courant demandé au buck 5 V | 1,5 A | ≈ 1,35 A | 1 A (buck séparé) |
| Perte propre à la voie 3,3 V | 0,85 W | ≈ 0,12 W | ≈ 0,36 W |
| Rendement global estimé | ≈ 78 % | ≈ 86 % | ≈ 86 % |
| Température de jonction à 60 °C ambiants | ≈ 111 °C (θJA = 60 °C/W) | ≈ 66 °C | faible |
| Marge avec déclassement (100 °C) | Négative | Correcte | Correcte |
| Complexité | Très simple | Moyenne | Moyenne |
| Composants haute tension à qualifier | 1 | 1 | 2 |
| Bruit sur le 3,3 V | Très faible | Ondulation de commutation (≈ 1,4 mV calculés) | Ondulation de commutation |

Les valeurs de l'option B viennent des calculs détaillés (étape 3).
Celles des options A et C sont des estimations simples (rendement du buck C
supposé à 82 %).

**Choix : option B (buck 5→3,3 V).** Le LDO ne passe pas le critère de
déclassement à 60 °C ambiants : sa température de jonction approche 110 °C
avec une bonne surface de cuivre. Le buck en cascade offre un rendement
acceptable, n'ajoute qu'un petit circuit basse tension et ne multiplie pas
les composants haute tension. L'option C est équivalente en rendement mais
double le nombre de convertisseurs 60 V à router et à valider. Le LDO reste
la bonne solution pour une alimentation analogique sensible au bruit, ce
qui n'est pas le cas ici.

Conséquence à vérifier en simulation : le buck 3,3 V est une charge à
puissance constante (|Zin| ≈ 14 Ω) pour le buck 5 V. L'impédance de sortie
du buck 5 V doit rester très inférieure (critère de Middlebrook).

## 4. Protection contre les défauts : fusible plutôt que PTC

**Choix : fusible CMS 1 A, fusion lente, 63 V DC.**
- Calibre : 0,5 A / (0,75 × 0,9) ≈ 0,74 A, donc 1 A normalisé
- Le courant de maintien d'un PTC baisse avec la température (60 °C
  ambiants), et si la TVS tombe en court-circuit, un PTC la laisse chauffer
- Le fusible isole la carte quand un composant d'entrée (TVS, MOSFET,
  condensateur) se met en court-circuit
- Il ne protège pas contre le court-circuit de sortie : chaque buck gère
  le sien par limitation de courant

## 5. Ordre des blocs

Connecteur 24 V → fusible → P-MOSFET anti-inversion → TVS →
condensateurs d'entrée → buck 24→5 V → buck 5→3,3 V → LED et points de test

La TVS est placée après la protection contre l'inversion : si elle était
avant, une inversion de polarité la ferait conduire en direct, ce qui
court-circuite l'alimentation et sollicite le fusible inutilement.

Aucun condensateur n'est placé en amont du MOSFET (pas de résonance avec
le câble en cas d'inversion).

## 6. Choix de composants issus du dimensionnement et du déclassement

| Composant | Choix | Raison |
|---|---|---|
| TVS | SMBJ33A (VRWM 33 V, Vc ≈ 53,3 V) | VRWM > 32 V ; Vc < 60 V (limite du buck) |
| Condensateurs d'entrée céramiques | 2 × 4,7 µF, **100 V**, X7R/X7S, boîtier 1210 | La TVS écrête à 53 V : des 50 V seraient dépassés |
| Condensateur de volume d'entrée | 47 µF, 63 V, électrolytique ou polymère | Amortit les transitoires (ESR) |
| Buck 24→5 V | Famille TPS54360B (60 V), non synchrone, 500 kHz | Marge de tension ; diode externe |
| Diode de roue libre | Schottky 60 V, boîtier SMC (ou SMB avec large cuivre) | Tj ≈ 90 °C en SMC contre ≈ 107 °C en SMB |
| Inductance 22 µH | Blindée, Irms ≥ 1,8 A, **Isat > limite de courant du circuit** | Évite la saturation en court-circuit |
| Résistances de contre-réaction (les deux bucks) | 0,1 % | Avec du 1 %, le pire cas dépasse ±3 % |
| Buck 5→3,3 V | Famille TPS62160 (synchrone), ≈ 2 MHz | Rendement ≈ 93 % |

## 7. Contraintes de placement (pour le routage)
- Placer le buck 3,3 V à l'écart de la diode du buck 24→5 V, de
  l'inductance 22 µH et du P-MOSFET (la diode atteint ≈ 90 °C)
- Boucle de commutation du buck 24→5 V la plus petite possible
- Nœud SW petit, feedback éloigné de l'inductance
- Éventuel circuit RC de dissipation sur le nœud SW (à simuler)

## 8. Limites connues
- **Surtension permanente** (> 36,7 V) non couverte : la TVS et le fusible
  sont sacrifiés. Une protection active serait nécessaire (V2)
- **Marge de la diode Schottky pendant un surge** : 11 % (60 V contre 53,3 V).
  Option retenue : routage soigné et circuit RC éventuel, à valider à
  l'étape 4. Passer à 100 V coûterait en rendement et en température
- **Marge de la TVS** : VRWM à 97 % de Vin max, faible par construction
- **Rendements et paramètres thermiques** issus d'hypothèses, à remplacer
  par les fiches techniques

## 9. Points ouverts
- Fixer Isat de l'inductance 22 µH et recalculer la perte DCR avec la
  valeur réelle
- Comparer le pic d'appel de courant à la mise sous tension au courant
  impulsionnel (Idm) du MOSFET ; ajouter une résistance de 1 Ω en série
  si nécessaire
- Valider en simulation : démarrage à 18, 24 et 32 V, échelon de charge,
  stabilité du buck 5 V avec le buck 3,3 V en charge, ringing du nœud SW
  pendant un surge
- Remplacer toutes les hypothèses par les valeurs des fiches techniques
- Pic d’appel de courant de 45 à 75 A : vérifier l’Idm du MOSFET (S8)

## 10. Historique des modifications
| Version | Modifications |
|---|---|
| 0.1 | Version initiale |
| 0.2 | Courant d'entrée corrigé (0,43 A) ; rendement du buck 3,3 V à 93 % ; Vds du MOSFET passé à 60 V ; fusible retenu à la place du PTC ; condensateurs d'entrée 100 V ; tableau des composants et contraintes de placement ajoutés |
