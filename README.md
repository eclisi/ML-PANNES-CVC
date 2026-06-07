# ML-PANNES-CVC

**Auteur : Jimmy SAMBIN**

## Anticiper les pannes CVC avant qu'elles ne surviennent

**ML-PANNES-CVC** est un systeme de prediction des pannes sur les equipements de Chauffage, Ventilation et Climatisation (CVC) destine aux **ingenieurs et techniciens d'automatisme**. Son objectif : transformer les donnees capteurs en alertes predictives, pour intervenir **avant** la defaillance et non apres.

### Pourquoi ce projet ?

Dans un batiment hospitalier ou tertiaire, une panne CVC n'est pas qu'un incident technique — c'est une rupture de service qui impacte directement les occupants. Or, la maintenance curative (intervenir apres la panne) coute 3 a 5 fois plus cher que la maintenance preventive, sans compter les consequences operationnelles.

Ce projet repond a une question simple mais critique : **quand cet equipement va-t-il tomber en panne, et faut-il intervenir cette semaine ?**

### Comment ca marche ?

Le systeme repose sur un **moteur d'apprentissage** base sur l'algorithme **Random Forest** (foret aleatoire) : 100 arbres de decision independants analyserent les donnees capteurs, puis votent a la majorite pour determiner si un equipement est en bon etat ou en defaut imminent.

**4 variables capteurs** sont analysees :

| Variable | Unite | Role |
|----------|-------|------|
| Temperature | degC | Ecart au regime nominal |
| Vibration | mm/s | Indicateur principal de defaut mecanique |
| Heures de fonctionnement | h | Usure cumulee |
| Age de l'equipement | ans | Vieillissement structurel |

**Importance des variables** : Vibration domine a **95,8%**, suivie de Temperature (2,8%), Heures (1,4%), Age (0%). La vibration est le signal d'alarme le plus fiable — conforme a la norme **ISO 10816-3**.

### Performances du modele

| Metrique | Valeur |
|----------|--------|
| Accuracy | 94,4% |
| Precision | 94,0% |
| Recall (sensibilite) | 95,1% |
| F1-score | 94,6% |
| Specificite | 93,6% |
| Arbres | 100 |
| Feuille minimale | 5 observations |

### Seuils d'alerte

| Niveau | Probabilite defaut | Action |
|--------|--------------------|--------|
| **CRITIQUE** | >= 80% | Intervention immediate |
| **ALARME** | 60-79% | Inspection sous 48h |
| **VIGILANCE** | 40-59% | Surveillance renforcee |
| **OK** | < 40% | Maintenance planifiee |

### 10 equipements hospitaliers surveilles

CVC-001 a CVC-010 — unite(s) de traitement d'air, CTA, ventilation double flux, climatisation split, pompe a chaleur.

---

## Structure du depot

```
ML-PANNES-CVC/
|-- README.md
|-- .gitignore
|
|-- graphiques/
|   |-- exploration_donnees.png        # Distribution des 4 variables capteurs
|   |-- matrice_confusion.png          # Matrice de confusion Normal/Defaut
|   |-- feature_importance.png         # Importance des 4 variables (Vibration 95,8%)
|   |-- frontiere_decision.png         # Frontiere de decision Temperature-Vibration
|   |-- convergence_rf.png             # Convergence erreur OOB vs nombre d'arbres
|   |-- heatmap_semaine.png            # Heatmap probabilite defaut par equipement/jour
|   |-- heatmap_S02-06-2026.png        # Semaine calme (7 critiques)
|   |-- heatmap_S09-06-2026.png        # Semaine canicule (11 critiques)
|   |-- heatmap_S16-06-2026.png        # Semaine post-canicule (12 critiques)
|   |-- heatmap_S23-06-2026.png        # Semaine post-maintenance (5 critiques)
|
|-- data/
|   |-- entrainement/
|   |   |-- pannes_cvc.xlsx            # 800 observations (dataset d'entrainement)
|   |   |-- description_dataset.txt    # Description detaillee des 6 colonnes + seuils ISO
|   |
|   |-- mesures-semaines/
|       |-- S02-06-2026_mesures.xlsx   # Semaine 02/06 : mesures capteurs 10 equipements
|       |-- S02-06-2026_resultats.xlsx # Semaine 02/06 : resultats prediction
|       |-- S09-06-2026_mesures.xlsx   # Semaine 09/06 : canicule
|       |-- S09-06-2026_resultats.xlsx
|       |-- S16-06-2026_mesures.xlsx   # Semaine 16/06 : post-canicule
|       |-- S16-06-2026_resultats.xlsx
|       |-- S2026-06-06_mesures.xlsx   # Semaine 06/06 : courant
|       |-- S2026-06-06_resultats.xlsx
|       |-- S23-06-2026_mesures.xlsx   # Semaine 23/06 : post-maintenance
|       |-- S23-06-2026_resultats.xlsx
|
|-- previsions/
    |-- rapport_S02-06-2026.txt        # Rapport semaine calme
    |-- rapport_S09-06-2026.txt        # Rapport semaine canicule
    |-- rapport_S16-06-2026.txt        # Rapport semaine post-canicule
    |-- rapport_S2026-06-06.txt        # Rapport semaine courant
    |-- rapport_S23-06-2026.txt        # Rapport semaine post-maintenance
```

---

## Utilisation

### 1. Entrainement du modele

Le moteur d'apprentissage construit un Random Forest a partir du dataset `pannes_cvc.xlsx` (800 observations). Chaque arbre est entraine sur un sous-ensemble aleatoire des donnees (bagging), puis les 100 arbres votent pour la classification finale.

### 2. Prediction hebdomadaire

Pour chaque semaine, le systeme :
1. Collecte les mesures capteurs des 10 equipements (Temperature, Vibration, Heures, Age)
2. Soumet les donnees au modele entraine
3. Calcule la probabilite de defaut pour chaque equipement, chaque jour
4. Genere un rapport textuel avec recommandations d'intervention
5. Produit une heatmap visuelle des risques par equipement/jour

### 3. Interpretation pour le technicien

Le rapport hebdomadaire classe les equipements par niveau de criticite :
- **CRITIQUE** : intervenir immediatement (probabilite defaut >= 80%)
- **ALARME** : inspecter sous 48h (60-79%)
- **VIGILANCE** : surveiller (40-59%)
- **OK** : maintenance planifiee (< 40%)

Exemple de resultat : CVC-007 affiche 90-98% de probabilite defaut toute la semaine (vibrations elevees) → **intervention immediate requise**.

---

## Cas d'usage : evolution saisonniere

| Semaine | Contexte | Equipements critiques | Observation |
|---------|----------|-----------------------|-------------|
| S02 (juin) | Regime nominal | 7 | Reference de base |
| S09 (juin) | Canicule | 11 | Surcharge thermique, vibrations augmentees |
| S16 (juin) | Post-canicule | 12 | Stress thermique differe, pannes tardives |
| S23 (juin) | Post-maintenance | 5 | Effet de la maintenance preventive, seul CVC-007 reste critique |

Ce suivi montre l'impact direct de la meteo et de la maintenance sur la fiabilite des equipements — un argument concret pour justifier les budgets de maintenance preventive.

---

## Format des donnees

### Dataset d'entrainement (pannes_cvc.xlsx)

| Colonne | Type | Plage | Description |
|---------|------|-------|-------------|
| Temperature_C | numerique | 15-95 degC | Temperature de fonctionnement |
| Vibration_mm_s | numerique | 0.1-10 mm/s | Niveau vibratoire (ISO 10816-3) |
| Heures_Fonctionnement | numerique | 0-50000 h | Heures de service cumulees |
| Age_ans | numerique | 0-30 ans | Anciennete de l'equipement |
| Etat | categoriel | Normal/Defaut | Classification cible |
| Type_Defaut | entier | 0-3 | 0=Normal, 1=Mecanique, 2=Electrique, 3=Thermique |

### Mesures hebdomadaires (xlsx)

Memes 4 colonnes capteurs (Temperature_C, Vibration_mm_s, Heures_Fonctionnement, Age_ans) pour 10 equipements, 5 jours (lundi-vendredi).

---

## Seuils ISO 10816-3 (reference)

| Zone | Vibration (mm/s) | Etat |
|------|-------------------|------|
| A | 0 - 2.8 | Neuf / acceptable |
| B | 2.8 - 7.1 | Surveillance |
| C | 7.1 - 11.2 | Alerte / intervention planifiee |
| D | > 11.2 | Danger / arret immediat |

---

## Technologies

- **Moteur d'apprentissage** : Random Forest (100 arbres, MinLeafSize=5)
- **Format de donnees** : xlsx (Excel)
- **Visualisation** : PNG (heatmaps, matrices de confusion, frontiers de decision)
- **Classification** : binaire (Normal / Defaut) avec probabilite
- **Norme de reference** : ISO 10816-3 (vibrations machines tournantes)

---

## Licence

Projet open source — usage libre pour la maintenance preventive des equipements CVC.