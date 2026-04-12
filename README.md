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

### Étape 1 — Exploration visuelle par histogrammes

Avant tout nettoyage, on trace les histogrammes de **toutes les variables numériques brutes**. Cette étape sert à justifier empiriquement chaque décision d'inclusion ou d'exclusion, plutôt que de simplement la décréter.

Conclusions tirées des histogrammes :

| Variable | Observation | Décision |
|---|---|---|
| `equilibrium_temp_k` | Histogramme très lacunaire (~25% de NA) | ❌ Exclue |
| `star_age_gyr` | Histogramme très lacunaire (~21% de NA) | ❌ Exclue |
| `n_stars`, `n_planets` | Barre écrasée sur 1, quasi-constantes | ❌ Exclues |
| `ra`, `dec` | Aucune structure interprétable, hors problématique | ❌ Exclues |
| `star_vmag` | Dépend de la distance à la Terre, pas intrinsèque | ❌ Exclue |
| `orbital_period_days` | Distribution en "queue de comète", outliers extrêmes | ⚠️ Transformation log |
| `semi_major_axis_au` | Idem | ⚠️ Transformation log |
| `planet_mass_earth` | Idem | ⚠️ Transformation log |
| `planet_radius_earth` | Idem | ⚠️ Transformation log |
| Autres variables actives | Distributions exploitables | ✅ Conservées |

### Étape 2 — Transformation logarithmique

Quatre variables présentent des distributions **très asymétriques** avec des valeurs extrêmes très éloignées de la majorité (ex : `orbital_period_days` va de 0,09 jours à 400 millions de jours). Ces outliers risquent de **polluer l'ACP** en tirant artificiellement un axe entier vers eux.

On applique `log1p(x)` = `log(x + 1)` sur ces variables. Le `+1` évite `log(0)` qui est indéfini.

**Effet concret sur `orbital_period_days` :**

| Valeur réelle (jours) | Après log1p |
|---|---|
| 1 | 0,69 |
| 100 | 4,61 |
| 10 000 | 9,21 |
| 400 000 000 | 19,81 |

Le log compresse les grandes valeurs et raisonne en **ordres de grandeur**. Après transformation, les histogrammes deviennent bien plus symétriques.

> **Note :** La transformation log et le centrage-réduction (`scale.unit`) sont complémentaires. Le log corrige la **forme** de la distribution ; le centrage-réduction harmonise les **échelles**. Un outlier extrême reste un outlier même après centrage-réduction — seul le log corrige vraiment le problème.

### Étape 3 — Sélection des variables et nettoyage ciblé

On sélectionne d'abord les variables pertinentes, **puis** on supprime les NA — et non l'inverse.

**Pourquoi cet ordre ?** Supprimer les NA sur les 20 colonnes initiales revient à perdre plus de 75% des observations, car les valeurs manquantes sont réparties sur de nombreuses colonnes. En ne travaillant que sur les 10 variables actives retenues, on minimise drastiquement la perte de lignes.

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
- **`ncp = 5`** : on conserve 5 dimensions pour l'exploration initiale
- **Variables illustratives qualitatives :** `planet_type` (n'entre pas dans le calcul, sert à colorier les graphiques)

#### `scale.unit = TRUE` — Centrage et réduction

Cette option centre et réduit chaque variable : **moyenne = 0, écart-type = 1**.

C'est indispensable ici car les variables ont des unités très hétérogènes (jours, kelvin, masses solaires, sans unité…). Sans cette option, l'ACP maximise la variance totale et se focalise mécaniquement sur les variables avec les plus grandes valeurs absolues — pas parce qu'elles sont plus corrélées, mais simplement parce que leurs chiffres sont plus grands.

| | Sans `scale.unit` | Avec `scale.unit = TRUE` |
|---|---|---|
| Matrice analysée | Covariance | **Corrélation** |
| Influence des unités | ✅ Oui (biais) | ❌ Non (neutre) |
| Variables dominantes | Celles avec grande variance brute | Toutes équivalentes |
| Recommandé ici | ❌ | ✅ |

### Critères de sélection des axes
Trois critères complémentaires sont utilisés :
1. **Valeur propre > 1** (critère de Kaiser)
2. **Inertie cumulée ≥ 60%** (ou 85% selon le critère retenu)
3. **Coude visuel sur l'éboulis** des valeurs propres

---

## 📦 Librairies R utilisées

```r
library(readxl)      # Import du fichier Excel
library(FactoMineR)  # ACP
library(factoextra)  # Visualisations (fviz_eig, fviz_pca_var, etc.)
library(corrplot)    # Matrice de corrélation
library(ggplot2)     # Histogrammes d'exploration
library(tidyr)       # Mise en format long (pivot_longer)
library(dplyr)       # Manipulation des données (select, etc.)
```

---

## 📄 Structure du code R

```
Rapport_Planet.Rmd
│
├── 1. Import des données
│   └── read_xlsx()
│
├── 2. Exploration visuelle — histogrammes bruts
│   ├── Histogrammes de toutes les variables numériques
│   └── Justification empirique des exclusions et transformations
│
├── 3. Transformations log
│   ├── log1p() sur orbital_period_days, semi_major_axis_au
│   │            planet_mass_earth, planet_radius_earth
│   └── Histogrammes après transformation (vérification)
│
├── 4. Sélection des variables et nettoyage ciblé
│   ├── 5 variables planète (actives)
│   ├── 5 variables étoile (actives)
│   ├── 3 variables qualitatives (illustratives)
│   └── na.omit() sur le sous-ensemble sélectionné uniquement
│
├── 5. Matrice de corrélation
│   └── corrplot() — détection de variables redondantes ou dégénérées
│
├── 6. ACP
│   └── PCA() avec scale.unit=TRUE et planet_type en quali.sup
│
├── 7. Eboulis des valeurs propres
│   └── fviz_eig() — choix du nombre de dimensions
│
├── 8. Contributions des variables
│   ├── res.pca$var$contrib
│   └── fviz_contrib() par axe (axes 1, 2, 3)
│
├── 9. Cercles de corrélation
│   └── fviz_pca_var() — plans (1,2) et (1,3)
│
└── 10. Graphe des individus
    ├── fviz_pca_ind() — coloré par planet_type
    └── fviz_pca_biplot() — variables + individus superposés
```

---

## ⚠️ Points d'attention

- **`print()` obligatoire dans les boucles** : en R Markdown, les graphiques générés dans une boucle `for` ne s'affichent pas sans `print()`.
- **`factoextra` doit être chargé** dans le chunk `setup` pour que les fonctions `fviz_*` soient disponibles.
- **Les variables actives doivent toutes être numériques** : `planet_name`, `host_star` et `planet_type` sont passées en supplémentaires qualitatives.
- **Appliquer les transformations log AVANT le nettoyage** : elles portent sur le dataset complet, pas seulement sur le sous-ensemble sélectionné.

---

*Projet réalisé dans le cadre du cours de Méthodes Factorielles — IUT Lyon 2, S4*