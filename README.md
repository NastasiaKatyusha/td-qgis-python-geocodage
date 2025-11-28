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
###
### L'objectif est, à partir d'une table non géocodée contenant les établissements scolaires marseillais, de trouver l'arrêt de tram ou métro le plus proche, ainsi que la distance.
### 
# 1️⃣ Préparation — Créer la table des établissements
## ✔ Ouvrez la console Python dans QGIS

Menu : Plugins → Python Console
#### → Dans l’onglet Éditeur (grande zone blanche en bas), collez puis exécutez :

```python
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
```

Vous obtenez une table attributaire vide appelée etablissements.

## 2️⃣ Ajouter les établissements

### Dans l’éditeur Python, collez :

```python

etabs = [
    {"nom": "Lycée Thiers", "adresse": "5 pl Lycée", "cp": "13001", "ville": "Marseille"},
    {"nom": "Université AMU - Sciences Économiques", "adresse": "14 r Puvis de Chavannes", "cp": "13001", "ville": "Marseille"},
    {"nom": "Campus Luminy", "adresse": "163 avenue de Luminy", "cp": "13288", "ville": "Marseille"},
    {"nom": "Lycée Jean Perrin", "adresse": "Rue Verdillon", "cp": "13010", "ville": "Marseille"}
]

layer = QgsProject.instance().mapLayersByName("etablissements")[0]
pr = layer.dataProvider()

features = []
for e in etabs:
    f = QgsFeature(layer.fields())
    for key, val in e.items():
        f[key] = val
    features.append(f)

pr.addFeatures(features)
layer.updateFields()
layer.updateExtents()

```

 ### La table contient maintenant 4 établissements, mais sans géométrie (normal).

## 3️⃣ Géocoder les établissements avec QBANO (installer l'extension de la BAN si vous ne l'avez pas déjà).

### Dans QGIS :

### Extensions → QBANO → BAN – Géocoder une adresse


Paramètres :

Adresse → adresse

Code postal → cp

Commune → ville

SCR de sortie : EPSG:3395

Exportez sous :

➡ etab_3395

Vous obtenez une couche géocodée, avec des points sur la carte.

## 4️⃣ Charger les stations de métro / tram (AMP)

### Téléchargez sur :
### https://data.ampmetropole.fr

### Jeu de données :
### ➡ Stations de métro et de tramway – Marseille

### Dans QGIS :

### Charger la couche

### Clic droit → Exporter → Enregistrer sous…

### SCR : EPSG:3395

### Nom de la nouvelle couche :
### ➡ arrets_tram_metro_3395

## 5️⃣ Script Python : trouver l’arrêt le plus proche

### Dans la console Python → onglet Éditeur, collez :
```python
from qgis.core import (
    QgsProject,
    QgsSpatialIndex,
    QgsFeatureRequest,
    QgsField,
    edit
)
from PyQt5.QtCore import QVariant

etab_layer = QgsProject.instance().mapLayersByName("etab_3395")[0]
stops_layer = QgsProject.instance().mapLayersByName("arrets_tram_metro_3395")[0]

ARRET_NAME_FIELD = "arret"

# Vérification du SCR
if etab_layer.crs() != stops_layer.crs():
    raise Exception("Les deux couches n'ont pas le même SCR !")

pr = etab_layer.dataProvider()

def add_field_if_missing(name, qtype):
    if etab_layer.fields().indexOf(name) == -1:
        pr.addAttributes([QgsField(name, qtype)])

add_field_if_missing("nom_arret", QVariant.String)
add_field_if_missing("dist_m", QVariant.Double)
add_field_if_missing("lt_200m", QVariant.Int)

etab_layer.updateFields()

idx_nom_arret = etab_layer.fields().indexOf("nom_arret")
idx_dist_m = etab_layer.fields().indexOf("dist_m")
idx_lt_200m = etab_layer.fields().indexOf("lt_200m")

index = QgsSpatialIndex(stops_layer.getFeatures())

with edit(etab_layer):
    for feat in etab_layer.getFeatures():
        geom = feat.geometry()
        if geom is None or geom.isEmpty():
            continue

        nearest_ids = index.nearestNeighbor(geom.asPoint(), 1)
        req = QgsFeatureRequest().setFilterFid(nearest_ids[0])
        nearest_feat = next(stops_layer.getFeatures(req))

        dist = geom.distance(nearest_feat.geometry())

        etab_layer.changeAttributeValue(feat.id(), idx_nom_arret, nearest_feat[ARRET_NAME_FIELD])
        etab_layer.changeAttributeValue(feat.id(), idx_dist_m, dist)
        etab_layer.changeAttributeValue(feat.id(), idx_lt_200m, 1 if dist < 200 else 0)

print("Terminé !")
```

### La couche etab_3395 est maintenant enrichie automatiquement avec :

### nom_arret : station la plus proche

### dist_m : distance en mètres

### lt_200m : 1 = à moins de 200 m / 0 = plus loin

## 6️⃣ Visualisation finale

Idées de mise en forme dans QGIS :

Symbologie vert (1) / rouge (0) sur lt_200m

Affichage de la concaténation dist_m, nom et nom_arret comme étiquette (avec tampon pour visibilité)

Ajout de buffers (200 m) pour vérifier visuellement

Export en carte PNG ou PDF
