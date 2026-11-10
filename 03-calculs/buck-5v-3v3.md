# Calculs : buck 5 V → 3,3 V

## 0. Hypothèses (à remplacer par les fiches techniques)
| Paramètre | Valeur supposée | Source à vérifier |
|---|---|---|
| Circuit intégré | TPS62160 ou équivalent (exemple) | Fiche technique |
| Tension de référence Vref | 0,8 V, tolérance ±1 % | Fiche technique |
| Rds(on) MOSFET haut / bas | 0,15 Ω / 0,10 Ω | Fiche technique |
| Fréquence de commutation | ≈ 2 MHz | Fiche technique |
| Temps de commutation (tr + tf) | 10 ns | Estimation |
| DCR de l'inductance | 0,12 Ω | Fiche de l'inductance |
| Pertes du circuit (commande, repos) | 30 mW | Estimation |
| Limite de courant du circuit | ≈ 2 A (valeur haute) | Fiche technique |
| Tension d'entrée | 4,85 à 5,15 V | Spécification (5 V ±3 %) |
| Courant de sortie de calcul | 0,5 A | Spécification |

## 1. Rapport cyclique (valeurs idéales)
| Vin | D = Vout / Vin | Temps de conduction (D / fsw) |
|---|---|---|
| 4,85 V | 0,680 | 340 ns |
| 5,00 V | 0,660 | 330 ns |
| 5,15 V | 0,641 | 320 ns |

Vérification : le rapport cyclique reste très inférieur à 100 % (pas de
risque de chute de tension, ou « dropout »), et le temps de conduction
minimal (320 ns) est largement supérieur au Ton min de la fiche (à confirmer).

## 2. Inductance
Ondulation visée : 30 % de Iout, au pire cas (Vin max = 5,15 V)
ΔI cible = 0,3 × 0,5 = 0,15 A

L = Vout × (1 − D) / (fsw × ΔI) = 3,3 × 0,359 / (2 000 000 × 0,15) ≈ 3,95 µH
→ Valeur normalisée retenue : 4,7 µH

Ondulation réelle avec 4,7 µH :
| Vin | ΔI | Courant crête (Iout + ΔI/2) |
|---|---|---|
| 4,85 V | 0,112 A | 0,556 A |
| 5,00 V | 0,119 A | 0,560 A |
| 5,15 V | 0,126 A | 0,563 A |

Courant efficace : Irms = √(Iout² + ΔI²/12) ≈ 0,501 A

Critères de choix :
- Isat ≥ limite de courant du circuit (hypothèse 2 A). Un courant crête
  normal de 0,56 A n'exigerait que 0,75 A, mais en cas de court-circuit
  le courant monte jusqu'à la limite, et l'inductance ne doit pas saturer
- Irms (échauffement) ≥ 0,65 A
- Inductance blindée, DCR faible
- Mode de faible charge : sous ΔI/2 ≈ 60 mA, le convertisseur passe en
  conduction discontinue (ou en économie d'énergie selon le circuit).
  Vérifier dans la fiche, et choisir le mode à ondulation faible si disponible.

## 3. Condensateur de sortie
Ondulation : ΔVout = ΔI / (8 × fsw × Cout) + ΔI × ESR

Choix : 1 × 22 µF, 10 V, X7R (tension nominale 3× la tension de sortie).
Capacité effective sous 3,3 V : hypothèse 50 % → ≈ 11 µF (on calcule avec 10 µF).

ΔVout = 0,126 / (8 × 2 000 000 × 10e-6) + 0,126 × 0,005
      ≈ 0,8 mV + 0,6 mV ≈ 1,4 mV (objectif : < 50 mV)

Réponse à un échelon de charge (10 % → 90 %, soit ΔIstep = 0,4 A) :
ΔV ≈ ΔIstep / (2π × fc × Cout), avec fc la bande passante de la boucle
(hypothèse 100 kHz, à lire ou simuler) :
ΔV ≈ 0,4 / (6,28 × 100 000 × 10e-6) ≈ 64 mV, soit 1,9 %

Cette valeur est approximative et dépend de la compensation du circuit.
À valider à l'étape 4.

## 4. Condensateur d'entrée
Courant efficace : Irms,in = Iout × √(D × (1 − D))
| Vin | Irms,in |
|---|---|
| 4,85 V | 0,233 A |
| 5,00 V | 0,237 A |
| 5,15 V | 0,240 A |

Choix : 1 × 10 µF, 16 V, X7R, au plus près des broches d'alimentation du circuit.
Capacité effective sous 5 V : hypothèse 50 % → ≈ 5 µF.
ΔVin = Iout × D × (1 − D) / (fsw × Cin) = 0,5 × 0,230 / (2 000 000 × 5e-6)
     ≈ 11,5 mV

Ce condensateur s'ajoute à la charge capacitive vue par le buck 24→5 V :
à prendre en compte pour sa stabilité (étape 4).

## 5. Contre-réaction
Vout = Vref × (1 + R1 / R2)
R2 = 10 kΩ ; R1 = 10 k × (3,3 / 0,8 − 1) = 31,25 kΩ
→ R1 = 31,2 kΩ (série E192, 0,1 %)
Vout nominal = 0,8 × (1 + 3,12) = 3,296 V (−0,12 %)

Pire cas (Vref ±1 %) :
| Résistances | Vout min | Vout max |
|---|---|---|
| 0,1 % | 3,258 V (−1,3 %) | 3,334 V (+1,0 %) |
| 1 % | 3,214 V (−2,6 %) | 3,380 V (+2,4 %) |

Conclusion : avec des résistances 1 %, il ne reste presque aucune marge dans
la tolérance de ±3 % pour la régulation de ligne et de charge. On retient des
**résistances 0,1 %**, comme pour le buck 5 V. Courant dans le diviseur :
0,8 V / 10 kΩ = 80 µA, négligeable.

## 6. Pertes et rendement (Iout = 0,5 A, Pout = 1,65 W)
| Perte | Formule | 4,85 V | 5,00 V | 5,15 V |
|---|---|---|---|---|
| MOSFET haut | Irms² × Rds(on) × D | 0,026 W | 0,025 W | 0,024 W |
| MOSFET bas | Irms² × Rds(on) × (1 − D) | 0,008 W | 0,009 W | 0,009 W |
| Commutation | ½ × Vin × Iout × (tr+tf) × fsw | 0,024 W | 0,025 W | 0,026 W |
| Inductance | Irms² × DCR | 0,030 W | 0,030 W | 0,030 W |
| Circuit (commande, repos) | estimation | 0,030 W | 0,030 W | 0,030 W |
| **Total** | | **0,118 W** | **0,119 W** | **0,119 W** |
| **Rendement** | Pout / (Pout + pertes) | **93,3 %** | **93,3 %** | **93,3 %** |

Conséquences sur le reste de la conception :
- Puissance d'entrée de ce bloc : 1,65 / 0,933 ≈ 1,77 W, soit 0,35 A sous 5 V
- Courant total demandé au buck 24→5 V : 1 A + 0,35 A ≈ 1,35 A. La valeur de
  calcul de 1,4 A reste valable (marge de 4 %)
- Rendement global estimé : (6,65 W) / (6,77 W / 0,877) ≈ 86 % à 24 V
- Courant d'entrée à 18 V : 6,77 W / 0,886 / 18 V ≈ 0,42 A (on garde 0,5 A
  pour le dimensionnement)

## 7. Thermique (température ambiante 60 °C)
| Élément | Puissance | θJA supposé | Élévation | Tj estimée | Limite déclassée |
|---|---|---|---|---|---|
| Circuit intégré | ≈ 0,09 W | 70 °C/W | ≈ 6 °C | ≈ 66 °C | 100 °C : OK |
| Inductance | 0,03 W | | négligeable | | |

Point d'attention : la température locale autour de ce circuit peut dépasser
60 °C si la diode du buck 24→5 V (≈ 90 °C estimé) est proche. Placer le buck
3,3 V à l'écart de la diode, de l'inductance 22 µH et du P-MOSFET.

## 8. Démarrage et séquencement
- Le buck 3,3 V démarre quand le 5 V dépasse son seuil de verrouillage
  sous-tension (UVLO, typiquement 2,5 à 3 V). Broche EN reliée à V5, ou
  diviseur d'enable si l'on veut démarrer plus haut
- Temps de démarrage progressif (soft-start) : à lire dans la fiche. Avec
  1 ms, le courant de charge de la sortie ≈ 22 µF × 3,3 V / 1 ms ≈ 70 mA,
  sans problème pour le 5 V
- Ordre : V24 → V5 → V3V3, sans besoin d'un autre séquencement

## 9. Stabilité vis-à-vis du buck 5 V (point à simuler)
Un convertisseur régulé est une charge à puissance constante : son
impédance d'entrée est négative.
|Zin| ≈ Vin² / Pin = 25 / 1,77 ≈ 14 Ω
L'impédance de sortie du buck 5 V doit rester très inférieure à cette
valeur sur toute la plage de fréquences (critère de Middlebrook). À vérifier
en simulation : démarrage conjoint et échelons de charge sur le 3,3 V.

## 10. Déclassement (partiel)
| Composant | Valeur max | Valeur réelle | Taux |
|---|---|---|---|
| Circuit intégré, tension d'entrée | 17 V (hypothèse) | 5,5 V (avec surtension) | 32 % |
| Condensateur d'entrée | 16 V | 5,5 V | 34 % |
| Condensateur de sortie | 10 V | 3,4 V | 34 % |
| Inductance, Irms | ≥ 0,65 A | 0,50 A | ≤ 77 % |
| Inductance, Isat | ≥ 2 A | 0,56 A (normal) | 28 % |

## 11. Points ouverts
- Remplacer toutes les hypothèses par les fiches techniques
- Définir une tolérance dynamique dans la spécification (voir ci-dessous)
- Simuler l'échelon de charge et l'interaction avec le buck 24→5 V (étape 4)
