# Architecture et choix de conception

## 1. Bilan de puissance
| Élément | Calcul | Résultat |
|---|---|---|
| Puissance utile V5 | 5 V × 1 A | 5 W |
| Puissance utile V3V3 | 3,3 V × 0,37 A | 1,221 W |
| Puissance utile totale | | 6,221 W |
| Courant d'entrée max (à 18 V) | ≈ 8,6 W / 18 V | ≈ 0,5 A |

Hypothèses de rendement (à remplacer par les fiches techniques) :
buck 24→5 V : 87 % ; buck 5→3,3 V : 90 % ; buck 24→3,3 V : 82 %.

## 2. Protection contre l'inversion de polarité

| Critère | Diode Schottky en série | P-MOSFET en série |
|---|---|---|
| Chute / résistance | Vf ≈ 0,5 V à 0,5 A | Rds(on) ≈ 40 mΩ |
| Perte à 0,5 A | ≈ 0,25 W | ≈ 10 mW (I²R) |
| Impact sur le rendement | ≈ 3 % | négligeable |
| Nombre de composants | 1 | 3 (MOSFET, zener, résistance) |
| Points d'attention | Tension inverse ≥ 40 V | Vgs max souvent ±20 V : une zener (≈ 12 V) doit limiter Vgs, car l'entrée monte à 32 V ; Vds ≥ 40 V |
| Coût | Très faible | Faible |

**Choix : P-MOSFET** avec écrêtage de grille. La perte est quasi nulle, et la
diode Schottky resterait une option acceptable pour ce courant. Le MOSFET
demande plus de soin de conception, ce qui est précisément ce qu'on veut
démontrer. À documenter : la tension de seuil et la tension Vgs réelle à
18 V et à 32 V.

## 3. Génération du 3,3 V

| Critère | A : LDO 5→3,3 V | B : buck 5→3,3 V (en cascade) | C : buck 24→3,3 V (parallèle) |
|---|---|---|---|
| Courant demandé au buck 5 V | 1,5 A | ≈ 1,37 A | 1 A (buck séparé) |
| Perte propre à la voie 3,3 V | 0,85 W | ≈ 0,18 W | ≈ 0,36 W |
| Rendement global estimé | ≈ 77 % | ≈ 85 % | ≈ 86 % |
| Température de jonction à 60 °C ambiants | ≈ 111 °C (θJA = 60 °C/W) | faible | faible |
| Marge avec déclassement (≈ 100 °C) | Négative | Correcte | Correcte |
| Complexité | Très simple | Moyenne | Moyenne |
| Composant haute tension à qualifier | 1 | 1 | 2 |
| Bruit sur le 3,3 V | Très faible | Ondulation de commutation | Ondulation de commutation |

**Choix : option B (buck 5→3,3 V).** Le LDO, que j'envisageais pour un
premier projet, ne passe pas le critère de déclassement à 60 °C ambiants :
sa température de jonction approche 110 °C avec une bonne surface de cuivre,
et dépasse la limite avec un cuivre réduit. Le buck en cascade rend le
rendement acceptable, n'ajoute qu'un petit circuit basse tension peu coûteux
et ne multiplie pas les composants haute tension. L'option C est équivalente
en rendement mais double le nombre de convertisseurs 60 V à router et à
valider. Le LDO reste la bonne solution pour une alimentation analogique
sensible au bruit, ce qui n'est pas le cas ici.

## 4. Ordre des blocs

Connecteur 24 V → fusible ou PTC → P-MOSFET anti-inversion → TVS →
condensateurs d'entrée → buck 24→5 V → buck 5→3,3 V → LED et points de test

La TVS est placée après la protection contre l'inversion : si elle était
avant, une inversion de polarité la ferait conduire en direct, ce qui
court-circuite l'alimentation et sollicite le fusible inutilement.

## 5. Points à traiter à l'étape 3
- Courant de saturation de l'inductance du buck 24→5 V (courant crête > 1,4 A)
- Calibre du fusible (≈ 1 A à fusion lente à valider, à cause de l'appel de
  courant dû aux condensateurs d'entrée)
- Choix de la zener de grille et vérification de Vgs à 18 V et 32 V
- Remplacement des rendements supposés par ceux des fiches techniques
