# Spécification : alimentation industrielle 24 V DC → 5 V / 3,3 V

## 1. Objectif
Alimentation basse tension destinée à alimenter la logique d'une carte
industrielle (microcontrôleur, interfaces, capteurs) à partir d'un bus 24 V DC.

## 2. Entrée
- Tension nominale : 24 V DC
- Plage de fonctionnement : 18 V à 32 V
- Courant d'entrée maximal : à calculer (étape 3)

## 3. Sorties
| Sortie | Tension | Courant max | Tolérance |
|--------|---------|-------------|-----------|
| V5     | 5 V     | 1 A         | ±3 %      |
| V3V3   | 3,3 V   | 0,5 A       | ±3 %      |

Note : la sortie 3,3 V étant dérivée du 5 V, le régulateur 5 V doit fournir
1,5 A au total.

## 4. Performances visées
- Ondulation de sortie : < 50 mV crête à crête
- Température ambiante maximale : 60 °C
- Rendement : à estimer et à documenter (étape 3)

## 5. Protections
- Inversion de polarité
- Surtension en entrée (diode TVS)
- Court-circuit en sortie
- Surintensité (fusible ou PTC)

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
  valeurs maximales

## 9. Critères de validation
Chaque valeur ci-dessus doit être démontrée par calcul et simulation
(étapes 3 et 4). Le projet n'est pas prototypé.
