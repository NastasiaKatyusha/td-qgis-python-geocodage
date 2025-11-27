# td-qgis-python-geocodage

## TD : Géocoder & analyser les distances aux stations de métro/tram avec QGIS + Python

## 🎯 Objectifs du TD

À l’issue du TD, vous saurez :

### - Créer une table attributaire dans QGIS depuis Python

### - Géocoder des adresses avec QBANO / BAN

### - Charger et manipuler une couche d’arrêts de métro/tram

### - Utiliser un script Python pour :

### - trouver l’arrêt le plus proche

### - calculer la distance en mètres

### - ajouter un indicateur “à moins de 200 m”

### - Visualiser les résultats dans QGIS

#1️⃣ Préparation — Créer la table des établissements
## ✔ Ouvrez la console Python dans QGIS

Menu : Plugins → Python Console
#### → Dans l’onglet Éditeur (grande zone blanche en bas), collez puis exécutez :

from qgis.core import QgsVectorLayer, QgsField, QgsFields, QgsFeature, QgsProject
from PyQt5.QtCore import QVariant

fields = QgsFields()
fields.append(QgsField("nom", QVariant.String))
fields.append(QgsField("adresse", QVariant.String))
fields.append(QgsField("cp", QVariant.String))
fields.append(QgsField("ville", QVariant.String))

layer = QgsVectorLayer("None", "etablissements", "memory")
pr = layer.dataProvider()
pr.addAttributes(fields)
layer.updateFields()

QgsProject.instance().addMapLayer(layer)


Vous obtenez une table attributaire vide appelée etablissements.
