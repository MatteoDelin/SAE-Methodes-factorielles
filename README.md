# 🪐 Projet Analyse des Exoplanètes — README

## Problématique

> **Dans quelle mesure les propriétés physiques des exoplanètes sont-elles corrélées aux caractéristiques de leur étoile hôte ?**

---

## 📁 Données

### Source
- **Fichier :** `PlanetDataSet.xlsx`
- **Feuille :** `nasa_exoplanet_intelligence`
- **Origine :** NASA Exoplanet Intelligence

### Dimensions initiales
- **6 150 lignes** (une par exoplanète)
- **20 colonnes** (variables planète + étoile + identifiants)

### Description des variables

#### Identifiants / Variables illustratives qualitatives
| Variable | Description |
|---|---|
| `planet_name` | Nom de la planète |
| `host_star` | Nom de l'étoile hôte |
| `planet_type` | Type de planète (7 catégories) |

#### Propriétés de la planète (variables actives)
| Variable | Description | Unité |
|---|---|---|
| `orbital_period_days` | Période orbitale | Jours |
| `planet_radius_earth` | Rayon de la planète | Rayons terrestres |
| `planet_mass_earth` | Masse de la planète | Masses terrestres |
| `orbital_eccentricity` | Excentricité orbitale | Sans unité |
| `semi_major_axis_au` | Demi-grand axe orbital | UA (Unités Astronomiques) |

#### Propriétés de l'étoile hôte (variables actives)
| Variable | Description | Unité |
|---|---|---|
| `star_temp_k` | Température de l'étoile | Kelvin |
| `star_radius_sun` | Rayon de l'étoile | Rayons solaires |
| `star_mass_sun` | Masse de l'étoile | Masses solaires |
| `star_metallicity` | Métallicité de l'étoile | dex |
| `star_surface_gravity` | Gravité de surface | log(cm/s²) |

#### Variables exclues de l'analyse
| Variable | Raison de l'exclusion |
|---|---|
| `equilibrium_temp_k` | ~25% de valeurs manquantes |
| `star_age_gyr` | ~21% de valeurs manquantes |
| `n_stars`, `n_planets` | Peu informatifs pour la problématique |
| `star_vmag` | Magnitude visuelle, non pertinente ici |
| `ra`, `dec` | Coordonnées célestes, hors problématique |

### Répartition des types de planètes
| Type | Effectif | % |
|---|---|---|
| Mini-Neptune | 2 148 | 34,9% |
| Gas Giant | 1 734 | 28,2% |
| Super-Earth | 1 185 | 19,3% |
| Neptune-like | 479 | 7,8% |
| Super-Jupiter | 324 | 5,3% |
| Sub-Earth | 230 | 3,7% |
| Unknown | 50 | 0,8% |

---

## 🔧 Pré-traitement des données

### Stratégie de nettoyage
Le nettoyage a été réalisé en **deux étapes** pour minimiser la perte d'observations :

1. **Sélection des variables** — on ne conserve que les 10 variables actives pertinentes pour la problématique + les 3 variables illustratives. Les colonnes très lacunaires (`equilibrium_temp_k`, `star_age_gyr`) sont exclues dès cette étape.

2. **Suppression des NA** — `na.omit()` est appliqué uniquement sur le sous-ensemble sélectionné, ce qui préserve beaucoup plus de lignes qu'un nettoyage global.

### Pourquoi pas un nettoyage global ?
Supprimer toutes les lignes ayant au moins 1 NA sur les 20 colonnes initiales revient à perdre plus de 75% des observations, car les valeurs manquantes sont réparties sur de nombreuses colonnes différentes. La sélection préalable des variables est donc indispensable.

---

## 📊 Méthode d'analyse : ACP (Analyse en Composantes Principales)

### Pourquoi l'ACP ?
| Méthode | Applicable ? | Raison |
|---|---|---|
| **ACP** | ✅ **Oui** | Toutes les variables actives sont quantitatives continues |
| AFC | ❌ Non | S'applique à un tableau de contingence (2 variables qualitatives) |
| ACM | ❌ Non | S'applique à des variables qualitatives multiples |

L'ACP est la méthode naturelle ici car elle permet d'explorer les **corrélations entre variables quantitatives** et de réduire la dimensionnalité, ce qui correspond directement à notre problématique.

### Paramètres de l'ACP
- **Librairie :** `FactoMineR` (R)
- **`scale.unit = TRUE`** : les variables sont centrées-réduites (indispensable car les unités sont très hétérogènes — jours, kelvin, masses solaires…)
- **`ncp = 5`** : on conserve 5 dimensions pour l'exploration initiale
- **Variables illustratives qualitatives :** `planet_type` (n'entre pas dans le calcul, sert à colorier les graphiques)

### Critères de sélection des axes
Trois critères complémentaires sont utilisés :
1. **Valeur propre > 1** (critère de Kaiser)
2. **Inertie cumulée ≥ 60%** (ou 85% selon le critère retenu)
3. **Coude sur l'éboulis** des valeurs propres

---

## 📦 Librairies R utilisées

```r
library(readxl)      # Import du fichier Excel
library(FactoMineR)  # ACP
library(factoextra)  # Visualisations (fviz_eig, fviz_pca_var, etc.)
library(corrplot)    # Matrice de corrélation
```

---

## 📄 Structure du code R

```
Rapport_Planet.Rmd
│
├── 1. Import des données
│   └── read_xlsx()
│
├── 2. Sélection des variables pertinentes
│   ├── 5 variables planète (actives)
│   ├── 5 variables étoile (actives)
│   └── 3 variables qualitatives (illustratives)
│
├── 3. Nettoyage ciblé
│   └── na.omit() sur le sous-ensemble sélectionné
│
├── 4. Analyse des corrélations
│   └── corrplot() — détection de variables redondantes ou dégénérées
│
├── 5. ACP
│   └── PCA() avec planet_type en quali.sup
│
├── 6. Eboulis des valeurs propres
│   └── fviz_eig() — choix du nombre de dimensions
│
├── 7. Contributions des variables
│   ├── res.pca$var$contrib
│   └── fviz_contrib() par axe (axes 1, 2, 3)
│
├── 8. Cercles de corrélation
│   └── fviz_pca_var() — plans (1,2) et (1,3)
│
└── 9. Graphe des individus
    ├── fviz_pca_ind() — coloré par planet_type
    └── fviz_pca_biplot() — variables + individus superposés
```

---

## ⚠️ Points d'attention

- **`print()` obligatoire dans les boucles** : en R Markdown, les graphiques générés dans une boucle `for` ne s'affichent pas sans `print()`.
- **`factoextra` doit être chargé** dans le chunk `setup` pour que les fonctions `fviz_*` soient disponibles.
- **Les variables actives doivent toutes être numériques** : `planet_name`, `host_star` et `planet_type` sont passées en supplémentaires qualitatives.

---

*Projet réalisé dans le cadre du cours de Méthodes Factorielles — IUT Lyon 2, S4*