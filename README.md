# 🪐 Projet Analyse des Exoplanètes — README

## Problématique

> **Dans quelle mesure les propriétés physiques des exoplanètes sont-elles corrélées aux caractéristiques de leur étoile hôte ?**

---

## 📁 Données

### Source
- **Fichier utilisé :** `PlanetDataSet__pretraitement_.xlsx` ← toujours celui-là
- **Feuille :** `nasa_exoplanet_intelligence`
- **Origine :** NASA Exoplanet Intelligence

### Pourquoi le dataset pré-traité et pas l'original ?

Les deux datasets contiennent **exactement les mêmes 6 150 planètes** avec les mêmes valeurs — seul l'ordre des lignes diffère. Le dataset pré-traité est une version **enrichie** de l'original : il ajoute 11 colonnes supplémentaires sans en supprimer aucune. L'original est donc strictement inclus dans le pré-traité. On utilise le pré-traité pour bénéficier des variables illustratives supplémentaires (`star_type`, `habitable_zone_flag`, etc.).

### Dimensions
- **6 150 lignes** (une par exoplanète)
- **31 colonnes** (20 originales + 11 ajoutées lors du pré-traitement)

---

### Description des variables

Les noms ont été traduits en français dès l'étape 2 du code via `rename()`.

#### Identifiants / Variables exclues
| Nom français | Nom original | Raison de l'exclusion |
|---|---|---|
| `nom_planete` | `planet_name` | Identifiant texte, pas une variable d'analyse |
| `etoile_hote` | `host_star` | Identifiant texte |
| `nb_etoiles` | `n_stars` | Quasi-constante (écrasée sur 1) |
| `nb_planetes` | `n_planets` | Quasi-constante |
| `installation_decouverte` | `disc_facility` | 73 modalités, trop fragmenté |
| `planete_controversee` | `controversial_flag` | Quasi-constante (99% de 0) |
| `decouverte_recente` | `is_recent_discovery` | Redondant avec `annee_decouverte` |
| `categorie_distance` | `dist_category` | Redondant avec `distance_terre_pc` |
| `categorie_periode` | `orbital_period_cat` | Redondant avec `periode_orbitale_jours` |
| `magnitude_visuelle` | `star_vmag` | Dépend de la distance à la Terre, pas intrinsèque |
| `ascension_droite` | `ra` | Coordonnées célestes, hors problématique |
| `declinaison` | `dec` | Coordonnées célestes, hors problématique |
| `temp_equilibre_k` | `equilibrium_temp_k` | ~25% de valeurs manquantes |
| `age_etoile_gyr` | `star_age_gyr` | ~21% de valeurs manquantes |

#### Propriétés de la planète (variables actives)
| Nom français | Nom original | Description | Unité | Transfo log ? |
|---|---|---|---|---|
| `periode_orbitale_jours` | `orbital_period_days` | Période orbitale | Jours | ✅ |
| `rayon_planete_terre` | `planet_radius_earth` | Rayon | Rayons terrestres | ✅ |
| `masse_planete_terre` | `planet_mass_earth` | Masse | Masses terrestres | ✅ |
| `excentricite_orbitale` | `orbital_eccentricity` | Excentricité orbitale | Sans unité | ❌ |
| `demi_grand_axe_ua` | `semi_major_axis_au` | Demi-grand axe | UA | ✅ |

#### Propriétés de l'étoile hôte (variables actives)
| Nom français | Nom original | Description | Unité | Transfo log ? |
|---|---|---|---|---|
| `temp_etoile_k` | `star_temp_k` | Température | Kelvin | ❌ |
| `rayon_etoile_soleil` | `star_radius_sun` | Rayon | Rayons solaires | ❌ |
| `masse_etoile_soleil` | `star_mass_sun` | Masse | Masses solaires | ❌ |
| `metallicite_etoile` | `star_metallicity` | Métallicité | dex | ❌ |
| `gravite_surface_etoile` | `star_surface_gravity` | Gravité de surface | log(cm/s²) | ❌ |

#### Variables illustratives quantitatives supplémentaires (quanti.sup)
| Nom français | Nom original | Description |
|---|---|---|
| `annee_decouverte` | `disc_year` | Année de découverte de la planète |
| `distance_terre_pc` | `dist_from_earth_pc` | Distance à la Terre en parsecs |

#### Variables illustratives qualitatives supplémentaires (quali.sup)
| Nom français | Nom original | Description | Nb modalités |
|---|---|---|---|
| `type_planete` | `planet_type` | Type de planète | 7 |
| `type_etoile` | `star_type` | Type spectral de l'étoile | 8 |
| `methode_decouverte` | `discovery_method` | Méthode de détection | 11 |
| `zone_habitable` | `habitable_zone_flag` | Planète en zone habitable ? | 2 |
| `systeme_multi_planetes` | `multi_planet_system` | Système multi-planètes ? | 2 |

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
Via `rename()` de `dplyr` immédiatement après l'import. Tout le code utilise exclusivement les noms français.

### Étape 2 — Exploration visuelle par histogrammes
Histogrammes de toutes les variables numériques brutes. Sert à justifier empiriquement chaque exclusion ou transformation, comme dans le TD météo avec `snow`.

| Variable | Observation | Décision |
|---|---|---|
| `temp_equilibre_k` | ~25% de NA | ❌ Exclue |
| `age_etoile_gyr` | ~21% de NA | ❌ Exclue |
| `nb_etoiles`, `nb_planetes` | Quasi-constantes | ❌ Exclues |
| `planete_controversee` | Quasi-constante (99% de 0) | ❌ Exclue |
| `ascension_droite`, `declinaison` | Hors problématique | ❌ Exclues |
| `magnitude_visuelle` | Non intrinsèque | ❌ Exclue |
| `periode_orbitale_jours` | Queue de comète | ⚠️ Log |
| `demi_grand_axe_ua` | Idem | ⚠️ Log |
| `masse_planete_terre` | Idem | ⚠️ Log |
| `rayon_planete_terre` | Idem | ⚠️ Log |

### Étape 3 — Transformation logarithmique (`log1p`)
`log1p(x)` = `log(x+1)`. Le +1 évite `log(0)` indéfini.

**Pourquoi ?** Certaines variables ont des distributions très asymétriques avec des outliers extrêmes (ex : `periode_orbitale_jours` va de 0,09 à 400 millions de jours). Sans transformation, ces outliers tireraient artificiellement un axe ACP vers eux. Le log compresse les grandes valeurs et raisonne en ordres de grandeur.

**Complémentarité avec `scale.unit` :** le log corrige la *forme* de la distribution ; le centrage-réduction corrige les *échelles*. Un outlier extrême reste un outlier même après centrage-réduction — seul le log corrige vraiment le problème.

### Étape 4 — Sélection des variables et nettoyage ciblé
Sélection d'abord → `na.omit()` ensuite. Supprimer les NA sur les 20+ colonnes initiales ferait perdre >75% des observations.

### Pourquoi `annee_decouverte` et `distance_terre_pc` en quanti.sup ?

Ces deux variables ne décrivent pas une propriété physique intrinsèque des planètes ou de leurs étoiles — elles décrivent les **conditions dans lesquelles la planète a été observée**. Les inclure en variables actives biaiserait l'ACP : les axes refléteraient en partie comment les planètes ont été découvertes, pas ce qu'elles sont vraiment.

En les passant en **quantitatives supplémentaires**, on les projette *après coup* sur les axes déjà construits. Cela sert à **détecter des biais d'observation** :

- Si `annee_decouverte` est très corrélée à un axe → les instruments récents détectent des types de planètes différents des anciens → biais temporel → à mentionner comme limite
- Si `distance_terre_pc` est très corrélée à un axe → on détecte mieux certains types de planètes à courte distance → biais de distance → à mentionner comme limite
- Si les deux sont proches de 0 sur tous les axes → ✅ pas de biais, la structure détectée est réelle

---

## 📊 Méthode d'analyse : ACP

### Pourquoi l'ACP ?
| Méthode | Applicable ? | Raison |
|---|---|---|
| **ACP** | ✅ Oui | Toutes les variables actives sont quantitatives continues |
| AFC | ❌ Non | Tableau de contingence (2 variables qualitatives) |
| ACM | ❌ Non | Variables qualitatives multiples |

### `scale.unit = TRUE` — Centrage et réduction
Centre et réduit chaque variable : **moyenne = 0, écart-type = 1**.

Sans cette option, les variables aux grandes valeurs absolues domineraient mécaniquement l'ACP, indépendamment de toute corrélation réelle.

| | Sans `scale.unit` | Avec `scale.unit = TRUE` |
|---|---|---|
| Matrice analysée | Covariance | Corrélation |
| Influence des unités | ✅ Oui (biais) | ❌ Non |
| Variables dominantes | Celles avec grande variance brute | Toutes équivalentes |

### Critères de choix du nombre d'axes
1. **Kaiser** : valeur propre > 1 (soit > 10% avec 10 variables)
2. **Inertie cumulée** : ≥ 60%
3. **Coude** : rupture de pente sur l'éboulis

**Résultats sur notre éboulis :**

| Dimension | % inertie | % cumulé |
|---|---|---|
| 1 | 40,1% | 40,1% |
| 2 | 16,9% | 57,0% |
| 3 | 14,1% | 71,1% |
| 4 | 9,5% | 80,6% |

- Kaiser → axes 1, 2 et 3 (tous > 10%)
- Inertie ≥ 60% → atteint à l'axe 3 (71,1%)
- Coude → entre axe 3 (14,1%) et axe 4 (9,5%)

**Conclusion : les 3 critères convergent vers 3 axes.** On se concentre sur le plan (1,2) pour la visualisation (57% d'inertie), en sachant que l'axe 3 (14,1%) apporte encore de l'information.

---

## 🔍 Comment comprendre les axes de l'ACP

C'est la partie la plus importante pour l'interprétation. Voici une explication pas à pas.

### Un axe = une direction dans laquelle les données varient le plus

L'ACP cherche les directions dans lesquelles les planètes sont le plus différentes les unes des autres. L'axe 1 est la direction de **maximum de variance** — celui où les planètes s'écartent le plus. L'axe 2 est la direction de maximum de variance **parmi celles perpendiculaires à l'axe 1**, etc.

### Lire le cercle de corrélation pour comprendre un axe

Chaque flèche = une variable active. La flèche pointe dans la direction où les valeurs élevées de cette variable se trouvent.

**Règle 1 — Direction d'une flèche :**
- Flèche qui pointe vers la droite → les individus à droite sur l'axe 1 ont des valeurs élevées pour cette variable
- Flèche qui pointe vers la gauche → les individus à gauche ont des valeurs élevées
- Flèche qui pointe vers le haut → les individus en haut sur l'axe 2 ont des valeurs élevées

**Règle 2 — Longueur d'une flèche :**
- Flèche longue (proche du bord du cercle) → variable bien représentée sur ce plan, on peut l'interpréter
- Flèche courte (proche du centre) → variable mal représentée, ne pas interpréter sur ce plan

**Règle 3 — Angle entre deux flèches :**
- Angle faible (flèches dans la même direction) → variables corrélées positivement
- Angle droit → variables indépendantes
- Angle proche de 180° (flèches opposées) → variables corrélées négativement

### Exemple concret avec votre dataset

Supposons que sur le cercle de corrélation :
- `rayon_etoile_soleil`, `masse_etoile_soleil`, `temp_etoile_k` pointent vers la droite sur l'axe 1
- `rayon_planete_terre`, `masse_planete_terre` pointent vers le haut sur l'axe 2

On peut alors dire :
- **L'axe 1 représente la "taille/puissance" de l'étoile** : plus on va vers la droite, plus l'étoile est grande, massive et chaude
- **L'axe 2 représente la "taille" de la planète** : plus on va vers le haut, plus la planète est grande et massive

### Lire le graphique des individus à partir des axes

Une fois les axes interprétés, chaque planète sur le graphique se lit comme :
- Une planète à **droite** → orbite autour d'une étoile grande/massive/chaude
- Une planète à **gauche** → orbite autour d'une étoile petite/légère/froide
- Une planète en **haut** → grande et massive
- Une planète en **bas** → petite et légère

Une Gas Giant en haut à droite = grosse planète autour d'une grosse étoile.
Une Sub-Earth en bas à gauche = petite planète autour d'une petite étoile.

**C'est ça la réponse à votre problématique** : si les types de planètes se regroupent en zones distinctes du graphique, et si ces zones correspondent à des caractéristiques stellaires précises (lues sur le cercle de corrélation), alors oui — les propriétés physiques des planètes sont corrélées aux caractéristiques de leur étoile hôte.

---

## 📈 Guide d'interprétation des parties 13, 14 et 15

### Partie 13 — Projection des variables illustratives

#### Variables quantitatives supplémentaires (`annee_decouverte`, `distance_terre_pc`)

`res.pca$quanti.sup$coord` donne leurs **coordonnées sur chaque axe**, assimilables à des corrélations (entre -1 et 1).

| Coordonnée | Interprétation |
|---|---|
| Proche de 0 | Pas de lien avec cet axe ✅ pas de biais |
| Proche de 1 ou -1 | Fort lien → biais potentiel ⚠️ |

**Ce qu'on écrit dans la note méthodologique :**
> *"Les variables `annee_decouverte` et `distance_terre_pc` ont été projetées en supplémentaires pour détecter d'éventuels biais d'observation. Leurs coordonnées sur les axes (tableau X) montrent des valeurs proches de [résultat réel], ce qui [confirme l'absence de biais / suggère un biais à mentionner comme limite]."*

**Ce qu'on écrit dans le rapport professionnel :**
> *"L'analyse vérifie que les résultats ne sont pas biaisés par les conditions d'observation : ni la date de découverte ni la distance à la Terre ne semblent structurer les groupes identifiés."* (si pas de biais)

#### Variables qualitatives supplémentaires

Pour chaque variable, on trace les individus colorés par modalité avec des ellipses de confiance à 95%.

**Règle de lecture :** deux modalités sont discriminées si leurs **ellipses ne se chevauchent pas**.

**`type_etoile`** — la plus importante pour la problématique
- Ellipses séparées sur l'axe 1 → ✅ le type spectral de l'étoile structure l'axe 1 → répond directement à la problématique
- Ce qu'on écrit : *"Les différents types spectraux d'étoiles (G, K, F, M) occupent des positions distinctes sur le premier axe, confirmant que cet axe reflète bien les caractéristiques de l'étoile hôte."*

**`type_planete`** — discrimine-t-il les axes ?
- Ellipses séparées sur l'axe 2 → ✅ les types de planètes ont des profils physiques distincts
- Ce qu'on écrit : *"Les différentes familles de planètes se regroupent dans des zones distinctes, ce qui confirme que leurs propriétés physiques les différencient de manière significative."*

**`zone_habitable`**
- Ellipses séparées → *"Les planètes situées dans la zone habitable de leur étoile présentent un profil orbital caractéristique qui les distingue des autres."*
- Ellipses confondues → *"La zone habitable ne suffit pas à distinguer les planètes sur ce plan factoriel."*

**`systeme_multi_planetes`**
- Ellipses séparées → *"Les planètes appartenant à des systèmes à plusieurs planètes ont un profil différent des planètes isolées."*

**`methode_decouverte`**
- Ellipses très séparées → ⚠️ biais de détection à mentionner comme limite : *"La méthode de détection structure en partie les axes, ce qui reflète les biais propres à chaque technique d'observation plutôt que des différences physiques intrinsèques."*

---

### Partie 14 — Graphique des individus

Chaque point = une planète, positionnée selon ses **scores factoriels** (coordonnées sur les axes). Les ellipses de confiance à 95% entourent 95% des individus d'un même type.

**Comment lire :**
1. Regarder d'abord le cercle de corrélation pour savoir ce que représentent les axes
2. Identifier quels groupes (couleurs) se séparent
3. Croiser : un groupe en haut à droite = planètes avec valeurs élevées pour les variables qui pointent en haut à droite

**Ce qu'on écrit dans la note méthodologique :**
> *"Le graphique des individus représente chaque exoplanète par ses scores factoriels sur les axes 1 et 2. Les ellipses de confiance (niveau 95%) sont calculées par type de planète et interprétées conjointement avec le cercle de corrélation."*

**Ce qu'on écrit dans le rapport professionnel :**
> *"La carte des exoplanètes révèle des regroupements clairs selon leur type : [décrire ce qu'on voit réellement]. Cette séparation confirme que les propriétés physiques des planètes sont bien liées aux caractéristiques de leur étoile hôte — chaque famille de planètes occupe une région caractéristique."*

---

### Partie 15 — Biplot

Superposition sur un même graphique des **individus** (points colorés) et des **variables** (flèches noires).

**Règles de lecture :**

| Situation | Interprétation |
|---|---|
| Individu dans la direction d'une flèche | Valeur élevée pour cette variable |
| Individu dans la direction opposée à une flèche | Valeur faible pour cette variable |
| Individu perpendiculaire à une flèche | Variable ne discrimine pas cet individu |
| Groupe de planètes dans la direction d'un groupe de flèches | Ces planètes sont caractérisées par ces variables |

**Ce qu'on écrit dans la note méthodologique :**
> *"Le biplot représente simultanément les scores factoriels des individus et les coordonnées des variables sur les axes (loadings). L'angle entre deux flèches est interprétable comme une corrélation : angle faible = corrélation positive, angle droit = indépendance, angle obtus = corrélation négative."*

**Ce qu'on écrit dans le rapport professionnel :**
> *"Cette représentation synthétique permet de lire en un seul graphique comment les caractéristiques physiques structurent les différentes familles de planètes. [Décrire un ou deux groupes concrets avec leurs variables associées]."*

---

### Synthèse : comment répondre à la problématique

La réponse se construit en croisant les parties 13 à 20 :

1. **Les axes sont-ils interprétables ?** (cercles de corrélation, parties 11 et 16) → quelles variables structurent chaque axe ?
2. **Les types d'étoiles se séparent-ils ?** (partie 13, `type_etoile`) → si oui, les axes captent bien les différences entre étoiles
3. **Les types de planètes se séparent-ils ?** (parties 14 et 19) → si oui, les planètes ont des profils physiques distincts
4. **Les flèches étoile et planète pointent-elles dans des directions cohérentes ?** (parties 15 et 20) → si oui, les propriétés planète et étoile sont bien corrélées
5. **L'axe 3 apporte-t-il une information complémentaire ?** (parties 16-20) → des groupes qui se chevauchaient sur (1,2) se séparent-ils sur (1,3) ?

Si les réponses à 2, 3 et 4 sont oui → **réponse à la problématique : oui, les propriétés physiques des exoplanètes sont corrélées aux caractéristiques de leur étoile hôte.**

---

### Parties 16 à 20 — Analyse de l'axe 3

#### Pourquoi analyser l'axe 3 ?

Les trois critères statistiques (Kaiser, inertie cumulée, coude) convergent tous vers **3 axes**. L'axe 3 capte **14,1% de l'inertie**, ce qui est comparable à l'axe 2 (16,9%). L'ignorer reviendrait à passer à côté d'une structure réelle dans les données. Les analyses sur les plans (1,3) et (2,3) sont donc aussi rigoureuses que celles sur le plan (1,2).

#### Comment interpréter l'axe 3 ?

La démarche est **identique** aux axes 1 et 2 :

1. `fviz_contrib` axe 3 (partie 17) → quelles variables contribuent le plus à cet axe ?
2. Cercles de corrélation (1,3) et (2,3) (partie 16) → dans quelle direction pointent ces variables ?
3. Variables illustratives sur plan (1,3) (partie 18) → certains groupes se séparent-ils différemment ?
4. Graphique des individus (1,3) (partie 19) → les types de planètes se distinguent-ils mieux ?
5. Biplot (1,3) (partie 20) → quelles variables caractérisent quels groupes ?

#### Ce que l'axe 3 peut révéler

L'axe 3 capte une structure **complémentaire et indépendante** des axes 1 et 2. Concrètement :

- Des groupes de planètes qui **se chevauchaient** sur le plan (1,2) peuvent se **séparer clairement** sur le plan (1,3) → l'axe 3 discrimine selon un critère différent
- Des variables qui contribuaient peu aux axes 1 et 2 peuvent **dominer l'axe 3** → elles portent une information que les premiers axes ne captaient pas (ex : `excentricite_orbitale` ou `metallicite_etoile`)
- Certaines variables illustratives peuvent mieux discriminer sur (1,3) que sur (1,2) → l'axe 3 peut être plus pertinent pour expliquer `zone_habitable` ou `systeme_multi_planetes` par exemple

#### Ce qu'on écrit dans la note méthodologique
> *"L'axe 3 (14,1% d'inertie) a été retenu conformément aux trois critères de sélection. Son analyse sur les plans (1,3) et (2,3) révèle que les variables [X et Y, à compléter d'après fviz_contrib axe 3] contribuent majoritairement à cet axe. Cet axe apporte une information complémentaire aux deux premiers en capturant [décrire en fonction du cercle de corrélation]. Des groupes qui se chevauchaient sur le plan (1,2) [se distinguent / ne se distinguent pas davantage] sur le plan (1,3)."*

#### Ce qu'on écrit dans le rapport professionnel
> *"Une troisième dimension de l'analyse met en évidence [décrire en langage courant ce que l'axe 3 représente — ex : 'la forme des orbites et la composition chimique des étoiles']. Certaines familles de planètes qui apparaissaient proches sur la première carte se distinguent plus clairement sur cette nouvelle représentation, enrichissant ainsi notre compréhension des liens entre planètes et étoiles hôtes."*

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
├── 1.  Import des données (dataset pré-traité, 31 colonnes)
│
├── 2.  Traduction des noms de variables en français (rename)
│
├── 3.  Exploration visuelle — histogrammes bruts
│   ├── Toutes les variables numériques
│   └── Justification empirique des exclusions et transformations
│
├── 4.  Transformations log (log1p)
│   ├── periode_orbitale_jours, demi_grand_axe_ua
│   ├── masse_planete_terre, rayon_planete_terre
│   └── Histogrammes après transformation (vérification)
│
├── 5.  Sélection des variables et nettoyage ciblé
│   ├── 10 variables actives
│   ├── 2 variables quanti.sup (annee_decouverte, distance_terre_pc)
│   ├── 5 variables quali.sup (type_planete, type_etoile, ...)
│   └── na.omit() sur le sous-ensemble sélectionné
│
├── 6.  Matrice de corrélation (corrplot)
│
├── 7.  ACP (PCA avec quanti.sup et quali.sup)
│
├── 8.  Eboulis des valeurs propres (fviz_eig)
│
├── 9.  Choix des axes retenus (commentaire des 3 critères)
│
├── 10. Contributions des variables par axe (fviz_contrib x3)
│
├── 11. Cercles de corrélation — plans (1,2) et (1,3)
│
├── 12. Graphique des variables — qualité de représentation (cos2)
│
├── 13. Projection des variables illustratives — Plan (1,2)
│   ├── quanti.sup : res.pca$quanti.sup$coord → détection de biais
│   └── quali.sup : fviz_pca_ind x5 avec ellipses → discrimination
│
├── 14. Graphique des individus — Plan (1,2)
│   └── fviz_pca_ind coloré par type_planete avec ellipses
│
├── 15. Biplot — Plan (1,2)
│
├── 16. Analyse de l'axe 3 — Cercles de corrélation (1,3) et (2,3)
│
├── 17. Contribution des variables à l'axe 3 (fviz_contrib axe 3)
│
├── 18. Projection des variables illustratives — Plan (1,3)
│   └── quali.sup : fviz_pca_ind x5 avec ellipses sur plan (1,3)
│
├── 19. Graphique des individus — Plan (1,3)
│   └── fviz_pca_ind coloré par type_planete avec ellipses
│
└── 20. Biplot — Plan (1,3)
```

---

## ⚠️ Points d'attention

- **`print()` obligatoire dans les boucles** : graphiques invisibles sans ça en R Markdown
- **`factoextra` doit être chargé** dans le chunk setup
- **Booléens à convertir en facteurs** : `zone_habitable` et `systeme_multi_planetes` doivent être `as.factor()` avant PCA
- **Ordre des colonnes dans `planet_clean`** : actives d'abord, quanti.sup ensuite, quali.sup en dernier — les indices `idx_quanti_sup` et `idx_quali_sup` en dépendent
- **Log AVANT le nettoyage** : les transformations s'appliquent sur le dataset complet
- **Renommage EN PREMIER** : tout le code utilise les noms français, jamais les noms originaux anglais
- **Ne pas interpréter les flèches courtes** sur le cercle de corrélation (mal représentées sur ce plan)
- **`ncp = 3` dans PCA()** : indispensable pour que l'axe 3 soit calculé et disponible pour les parties 16-20. Avec `ncp = 2`, les graphiques des plans (1,3) et (2,3) planteraient
- **Les plans (1,3) et (2,3) sont indépendants** : on ne peut pas "lire" le plan (1,3) sans avoir d'abord compris ce que représente l'axe 1 grâce au plan (1,2)

---

*Projet réalisé dans le cadre du cours de Méthodes Factorielles — IUT Lyon 2, S4*