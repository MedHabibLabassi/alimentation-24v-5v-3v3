# Spécification : alimentation industrielle 24 V DC → 5 V / 3,3 V

Version 0.2

## 1. Objectif
Alimentation basse tension destinée à alimenter la logique d'une carte
industrielle (microcontrôleur, interfaces, capteurs) à partir d'un bus 24 V DC.

## 2. Entrée
- Tension nominale : 24 V DC
- Plage de fonctionnement : 18 V à 32 V
- Courant d'entrée maximal : ≈ 0,43 A à 18 V (calcul étape 3).
  Valeur de dimensionnement retenue : 0,5 A

## 3. Sorties
| Sortie | Tension | Courant max | Tolérance statique |
|--------|---------|-------------|--------------------|
| V5     | 5 V     | 1 A         | ±3 %               |
| V3V3   | 3,3 V   | 0,5 A       | ±3 %               |

Notes :
- La tolérance statique inclut la précision de la référence, des résistances
  de contre-réaction, la régulation de ligne et la régulation de charge.
- La sortie 3,3 V est dérivée du 5 V (buck en cascade). Le régulateur 5 V
  doit donc fournir environ 1,35 A au total (1 A + ≈ 0,35 A pour le buck
  3,3 V). Valeur de calcul retenue : 1,4 A.
- Tolérance dynamique (échelon de charge 10 % → 90 %) : ±5 %.

## 4. Performances visées
- Ondulation de sortie : < 50 mV crête à crête
- Température ambiante maximale : 60 °C
- Température de jonction maximale de chaque composant (déclassée) : 100 °C
- Rendement global estimé (étape 3, à confirmer par simulation) :
  ≈ 86,6 % à 18 V, ≈ 85,9 % à 24 V, ≈ 85,0 % à 32 V, à pleine charge

## 5. Protections
- Inversion de polarité (P-MOSFET en série)
- Surtension transitoire en entrée (diode TVS)
- Court-circuit en sortie (limitation de courant des convertisseurs)
- Défaut des composants d'entrée (fusible 1 A, fusion lente)
- Surge de dimensionnement : 9 A, 10/1000 µs (pas de conformité normative
  revendiquée)

Limite connue : la TVS protège contre les transitoires, pas contre une
surtension permanente supérieure à environ 36,7 V. Dans ce cas, la TVS et
le fusible sont sacrifiés pour protéger la carte.

## 6. Diagnostic
- LED de présence pour chaque tension de sortie
- Points de test : 24 V, 5 V, 3,3 V, GND

## 7. Contraintes mécaniques
- Entrée : bornier à vis 5,08 mm
- Sorties : bornier ou connecteur (à définir)
- Dimensions : à définir
- Carte 2 couches

## 8. Contraintes de conception
- Composants courants, disponibles chez plusieurs distributeurs
  (Mouser, Digikey, LCSC)
- Déclassement : chaque composant utilisé à moins de 70-80 % de ses
  valeurs maximales en régime permanent
- Pendant un transitoire écrêté par la TVS (≈ 53 V), toute marge inférieure
  à 15 % est signalée dans le tableau de déclassement

## 9. Critères de validation
Chaque valeur ci-dessus doit être démontrée par calcul et simulation
(étapes 3 et 4). Le projet n'est pas prototypé.

## 10. Historique des modifications
| Version | Modifications |
|---|---|
| 0.1 | Version initiale |
| 0.2 | Courant d'entrée (0,43 A) et courant du buck 5 V (1,4 A) précisés ; tolérance dynamique ±5 % ; surge de dimensionnement ; fusible 1 A ; limite connue (surtension permanente) ; rendements estimés |
