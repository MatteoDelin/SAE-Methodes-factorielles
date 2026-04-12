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

Les noms de colonnes ont été traduits en français dès l'étape 2 du code via `rename()`.

#### Identifiants / Variables illustratives qualitatives
| Nom français | Nom original | Description |
|---|---|---|
| `nom_planete` | `planet_name` | Nom de la planète |
| `etoile_hote` | `host_star` | Nom de l'étoile hôte |
| `type_planete` | `planet_type` | Type de planète (7 catégories) |

#### Propriétés de la planète (variables actives)
| Nom français | Nom original | Description | Unité |
|---|---|---|---|
| `periode_orbitale_jours` | `orbital_period_days` | Période orbitale | Jours |
| `rayon_planete_terre` | `planet_radius_earth` | Rayon de la planète | Rayons terrestres |
| `masse_planete_terre` | `planet_mass_earth` | Masse de la planète | Masses terrestres |
| `excentricite_orbitale` | `orbital_eccentricity` | Excentricité orbitale | Sans unité |
| `demi_grand_axe_ua` | `semi_major_axis_au` | Demi-grand axe orbital | UA |

#### Propriétés de l'étoile hôte (variables actives)
| Nom français | Nom original | Description | Unité |
|---|---|---|---|
| `temp_etoile_k` | `star_temp_k` | Température de l'étoile | Kelvin |
| `rayon_etoile_soleil` | `star_radius_sun` | Rayon de l'étoile | Rayons solaires |
| `masse_etoile_soleil` | `star_mass_sun` | Masse de l'étoile | Masses solaires |
| `metallicite_etoile` | `star_metallicity` | Métallicité de l'étoile | dex |
| `gravite_surface_etoile` | `star_surface_gravity` | Gravité de surface | log(cm/s²) |

#### Variables exclues de l'analyse
| Nom français | Nom original | Raison de l'exclusion |
|---|---|---|
| `temp_equilibre_k` | `equilibrium_temp_k` | ~25% de valeurs manquantes |
| `age_etoile_gyr` | `star_age_gyr` | ~21% de valeurs manquantes |
| `nb_etoiles`, `nb_planetes` | `n_stars`, `n_planets` | Quasi-constantes (variance trop faible) |
| `magnitude_visuelle` | `star_vmag` | Dépend de la distance à la Terre, pas intrinsèque |
| `ascension_droite`, `declinaison` | `ra`, `dec` | Coordonnées célestes, hors problématique |

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

### Étape 1 — Traduction des noms de variables

Les noms de colonnes sont traduits en français via `rename()` de `dplyr`, immédiatement après l'import. Toute la suite du code utilise exclusivement les noms français.

### Étape 2 — Exploration visuelle par histogrammes

Avant tout nettoyage, on trace les histogrammes de **toutes les variables numériques brutes**. Cette étape sert à justifier empiriquement chaque décision d'inclusion ou d'exclusion, plutôt que de simplement la décréter.

Conclusions tirées des histogrammes :

| Variable | Observation | Décision |
|---|---|---|
| `temp_equilibre_k` | Histogramme très lacunaire (~25% de NA) | ❌ Exclue |
| `age_etoile_gyr` | Histogramme très lacunaire (~21% de NA) | ❌ Exclue |
| `nb_etoiles`, `nb_planetes` | Barre écrasée sur 1, quasi-constantes | ❌ Exclues |
| `ascension_droite`, `declinaison` | Aucune structure interprétable, hors problématique | ❌ Exclues |
| `magnitude_visuelle` | Dépend de la distance à la Terre, pas intrinsèque | ❌ Exclue |
| `periode_orbitale_jours` | Distribution en "queue de comète", outliers extrêmes | ⚠️ Transformation log |
| `demi_grand_axe_ua` | Idem | ⚠️ Transformation log |
| `masse_planete_terre` | Idem | ⚠️ Transformation log |
| `rayon_planete_terre` | Idem | ⚠️ Transformation log |
| Autres variables actives | Distributions exploitables | ✅ Conservées |

### Étape 3 — Transformation logarithmique

Quatre variables présentent des distributions **très asymétriques** avec des valeurs extrêmes très éloignées de la majorité (ex : `periode_orbitale_jours` va de 0,09 jours à 400 millions de jours). Ces outliers risquent de **polluer l'ACP** en tirant artificiellement un axe entier vers eux.

On applique `log1p(x)` = `log(x + 1)` sur ces variables. Le `+1` évite `log(0)` qui est indéfini.

**Effet concret sur `periode_orbitale_jours` :**

| Valeur réelle (jours) | Après log1p |
|---|---|
| 1 | 0,69 |
| 100 | 4,61 |
| 10 000 | 9,21 |
| 400 000 000 | 19,81 |

Le log compresse les grandes valeurs et raisonne en **ordres de grandeur**. Après transformation, les histogrammes deviennent bien plus symétriques.

> **Note :** La transformation log et le centrage-réduction (`scale.unit`) sont complémentaires. Le log corrige la **forme** de la distribution ; le centrage-réduction harmonise les **échelles**. Un outlier extrême reste un outlier même après centrage-réduction — seul le log corrige vraiment le problème.

### Étape 4 — Sélection des variables et nettoyage ciblé

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
- **Variables illustratives qualitatives :** `type_planete` (n'entre pas dans le calcul, sert à colorier les graphiques)

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
library(dplyr)       # Manipulation des données (rename, select, etc.)
```

---

## 📄 Structure du code R

```
Rapport_Planet.Rmd
│
├── 1. Import des données
│   └── read_xlsx()
│
├── 2. Traduction des noms de variables en français
│   └── rename() — 20 colonnes renommées
│
├── 3. Exploration visuelle — histogrammes bruts
│   ├── Histogrammes de toutes les variables numériques
│   └── Justification empirique des exclusions et transformations
│
├── 4. Transformations log
│   ├── log1p() sur periode_orbitale_jours, demi_grand_axe_ua
│   │            masse_planete_terre, rayon_planete_terre
│   └── Histogrammes après transformation (vérification)
│
├── 5. Sélection des variables et nettoyage ciblé
│   ├── 5 variables planète (actives)
│   ├── 5 variables étoile (actives)
│   ├── 3 variables qualitatives (illustratives)
│   └── na.omit() sur le sous-ensemble sélectionné uniquement
│
├── 6. Matrice de corrélation
│   └── corrplot() — détection de variables redondantes ou dégénérées
│
├── 7. ACP
│   └── PCA() avec scale.unit=TRUE et type_planete en quali.sup
│
├── 8. Eboulis des valeurs propres
│   └── fviz_eig() — choix du nombre de dimensions
│
├── 9. Contributions des variables
│   ├── res.pca$var$contrib
│   └── fviz_contrib() par axe (axes 1, 2, 3)
│
├── 10. Cercles de corrélation
│    └── fviz_pca_var() — plans (1,2) et (1,3)
│
└── 11. Graphe des individus
    ├── fviz_pca_ind() — coloré par type_planete
    └── fviz_pca_biplot() — variables + individus superposés
```

---

## ⚠️ Points d'attention

- **`print()` obligatoire dans les boucles** : en R Markdown, les graphiques générés dans une boucle `for` ne s'affichent pas sans `print()`.
- **`factoextra` doit être chargé** dans le chunk `setup` pour que les fonctions `fviz_*` soient disponibles.
- **Les variables actives doivent toutes être numériques** : `nom_planete`, `etoile_hote` et `type_planete` sont passées en supplémentaires qualitatives.
- **Appliquer les transformations log AVANT le nettoyage** : elles portent sur le dataset complet, pas seulement sur le sous-ensemble sélectionné.
- **Le renommage est fait en premier** : toute référence à une variable dans le code utilise le nom français, jamais le nom original anglais.

---

*Projet réalisé dans le cadre du cours de Méthodes Factorielles — IUT Lyon 2, S4*