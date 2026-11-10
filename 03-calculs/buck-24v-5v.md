# Calculs : buck 24 V → 5 V

## 0. Hypothèses (à remplacer par les fiches techniques)
| Paramètre | Valeur supposée | Source à vérifier |
|---|---|---|
| Circuit intégré | TPS54360B (exemple) | Fiche technique TI |
| Tension de référence Vref | 0,8 V, tolérance ±1,5 % | Fiche technique |
| Rds(on) du MOSFET haut | 0,1 Ω | Fiche technique |
| Fréquence de commutation | 500 kHz | Choix de conception |
| Diode de roue libre | Schottky 60 V, Vf ≈ 0,5 V | Fiche technique |
| Temps de commutation (tr + tf) | 20 ns | Estimation |
| DCR de l'inductance | 0,1 Ω | Fiche technique |
| Courant de sortie de calcul | 1,4 A | Étape 2 |

Remarque : le TPS54360B est, à ma connaissance, un buck non synchrone qui
exige une diode externe. À confirmer dans la fiche technique.

## 1. Rapport cyclique (valeurs idéales, sans les chutes)
D = Vout / Vin
| Vin | D | Temps de conduction (D / fsw) |
|---|---|---|
| 18 V | 0,278 | 556 ns |
| 24 V | 0,208 | 417 ns |
| 32 V | 0,156 | 313 ns |

Vérification : le temps de conduction minimal (313 ns à 32 V) doit rester
supérieur au minimum de la fiche technique (Ton min).

## 2. Inductance
Ondulation visée : 30 % de Iout, au pire cas (Vin max = 32 V)
ΔI cible = 0,3 × 1,4 = 0,42 A

L = Vout × (1 − D) / (fsw × ΔI)
L = 5 × (1 − 0,156) / (500 000 × 0,42) = 20,1 µH
→ Valeur normalisée retenue : 22 µH

Ondulation réelle avec 22 µH :
| Vin | ΔI | Courant crête (Iout + ΔI/2) |
|---|---|---|
| 18 V | 0,33 A | 1,56 A |
| 24 V | 0,36 A | 1,58 A |
| 32 V | 0,38 A | 1,59 A |

Courant efficace : Irms = √(Iout² + ΔI²/12) = √(1,96 + 0,012) ≈ 1,40 A

Critères de choix :
- Isat ≥ 1,3 × 1,59 ≈ 2,1 A (le courant ne doit jamais saturer l'inductance)
- Contrôler aussi la limite de courant du circuit : en cas de court-circuit,
  le courant monte jusqu'à cette limite, et l'inductance doit la supporter
- Irms (échauffement) ≥ 1,8 A pour garder de la marge
- Inductance blindée (moins de rayonnement)
- Point de fonctionnement en charge légère : en dessous de ≈ 0,2 A, le
  convertisseur passe en mode de faible charge (vérifier dans la fiche)

## 3. Condensateur de sortie
Ondulation : ΔVout = ΔI / (8 × fsw × Cout) + ΔI × ESR

Capacité minimale pour 20 mV (part capacitive) :
Cout = 0,38 / (8 × 500 000 × 0,020) = 4,8 µF effectifs

Choix : 2 × 22 µF, 16 V, céramique X7R/X5R.
Perte de capacité sous polarisation : hypothèse 50 % à 5 V
(à vérifier avec la courbe du fabricant) → ≈ 22 µF effectifs.

ΔVout = 0,38 / (8 × 500 000 × 22e-6) + 0,38 × 0,005
      ≈ 4,4 mV + 1,9 mV ≈ 6 mV (objectif : < 50 mV)

Cette valeur ne contient pas les pointes de commutation (inductance parasite
des pistes), qui dépendent du routage. La valeur finale du condensateur de
sortie dépend aussi de la stabilité de la boucle : à valider avec WEBENCH
(étape 4).

## 4. Condensateurs d'entrée
Courant efficace : Irms,in = Iout × √(D × (1 − D))
| Vin | Irms,in |
|---|---|
| 18 V | 0,63 A (pire cas) |
| 24 V | 0,57 A |
| 32 V | 0,51 A |

Choix : 2 × 4,7 µF, 50 V, X7R, au plus près du circuit.
Capacité effective supposée à 24 V : 50 % → ≈ 5 µF au total.
ΔVin = Iout × D × (1 − D) / (fsw × Cin) = 1,4 × 0,2006 / (500 000 × 5e-6)
     ≈ 0,11 V au pire cas (18 V)

En plus : un condensateur de volume (47 µF, 50 V, électrolytique ou polymère)
à l'entrée. Il amortit les transitoires et les oscillations dues à
l'inductance des câbles d'alimentation.

## 5. Contre-réaction
Vout = Vref × (1 + R1 / R2)
R2 = 10 kΩ ; R1 = 10 k × (5 / 0,8 − 1) = 52,5 kΩ → R1 = 52,3 kΩ (série E96)
Vout nominal = 0,8 × (1 + 5,23) = 4,984 V (−0,3 %)

Pire cas (Vref ±1,5 %) :
| Résistances | Vout min | Vout max |
|---|---|---|
| 1 % | 4,83 V (−3,4 %) | 5,15 V (+2,9 %) |
| 0,1 % | 4,90 V (−2,0 %) | 5,07 V (+1,3 %) |

Conclusion : avec des résistances 1 %, le pire cas dépasse la tolérance de
±3 % de la spécification, avant même la régulation de charge. On choisit donc
des **résistances 0,1 %** pour R1 et R2.

## 6. Pertes et rendement (Iout = 1,4 A, Pout = 7 W)
| Perte | Formule | 18 V | 24 V | 32 V |
|---|---|---|---|---|
| Diode de roue libre | Vf × Iout × (1 − D) | 0,505 W | 0,554 W | 0,591 W |
| Conduction MOSFET | Irms² × Rds(on) × D | 0,055 W | 0,041 W | 0,031 W |
| Commutation | ½ × Vin × Iout × (tr+tf) × fsw | 0,126 W | 0,168 W | 0,224 W |
| Inductance | Irms² × DCR | 0,197 W | 0,197 W | 0,197 W |
| Fonctionnement du circuit | estimation | 0,02 W | 0,025 W | 0,03 W |
| **Total** | | **0,90 W** | **0,99 W** | **1,07 W** |
| **Rendement** | Pout / (Pout + pertes) | **88,6 %** | **87,7 %** | **86,7 %** |

Constat : la diode représente plus de la moitié des pertes. Un circuit
synchrone (sans diode externe) améliorerait nettement le rendement : option
à étudier en version 2.

## 7. Thermique (température ambiante 60 °C)
| Composant | Puissance | θJA supposé | Élévation | Tj estimée | Limite (déclassée) |
|---|---|---|---|---|---|
| Circuit intégré (pire cas, 32 V) | 0,285 W | 50 °C/W | 14 °C | ≈ 74 °C | 100 °C |
| Diode, boîtier SMB | 0,59 W | 80 °C/W | 47 °C | ≈ 107 °C | 100 °C : trop chaud |
| Diode, boîtier SMC avec cuivre | 0,59 W | 50 °C/W | 30 °C | ≈ 90 °C | 100 °C : OK |

Décision : diode en boîtier SMC (ou SMB avec un large cuivre de dissipation).
Les valeurs de θJA sont des hypothèses : à remplacer par celles des fiches.

## 8. Choix de la diode
- Tension inverse : 60 V (la TVS écrête à environ 53 V lors d'une surtension)
- Courant moyen : Iout × (1 − D) = 1,18 A au pire cas → boîtier 3 A
- Schottky à faible fuite (la fuite augmente avec la température)

## 9. Points ouverts
- Remplacer les hypothèses par les valeurs réelles des fiches techniques
- Comparer ces calculs avec WEBENCH et LTspice (étape 4)
- Valider la stabilité de la boucle (compensation)
