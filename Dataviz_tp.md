# Synthèse — Mini TP Dataviz : analyse de parties de flipper

## Contexte & objectif global
Ce mini-TP simule une situation “data” réaliste : **analyser un flux de parties de flipper** (réelles ou simulées) pour :
- **évaluer la qualité d’exploitation** (taux d’échec / incidents),
- **comparer des outils** (zones d’arcade, types de machines),
- **prioriser des actions** (maintenance, réglage, surveillance),
- et **produire des visualisations** utiles à la décision.

**Dataset** : `data/flipper_games.csv`  


**Ressources / outils utilisés**
- `pandas` : chargement, nettoyage, 
- `numpy` : opérations numériques, mathématiques
- `seaborn` + `matplotlib` : barplot & boxplots pour l’analyse visuelle

## Chargement et contrôle

### Objetif
Avant toute dataviz, il faut **comprendre les données** :
- structure (lignes/colonnes),
- types de variables 
- volume (taille du dataset),

### Action
- Chargement du CSV dans `games`
- Affichage d’un `head()`
- Vérification des colonnes (`games.columns`, `info()`)
- Comptage des dimensions
- Comptage des joueurs uniques

### Résultat clé
- Le dataset contient **5409 lignes** et **14 colonnes**
- Nombre de joueurs uniques : **260**

### Interprétations
- Le dataset est suffisamment grand pour des agrégations par zone / machine.
Les paramètres du dataset permettront à analyser la qualité/risque du flipper

## Nettoyage minimal

### But

Ici, l’objectif est de retirer les valeurs clairement invalides avec :

- durées négatives ou nulles : filtrage `game_duration_s > 0`
- risques tilt hors bornes (0–100) : filtrage `tilt_risk_pct` dans `[0, 100]`
- pauses négatives : filtrage `pause_s >= 0`
- doublons éventuels.

## Indicateurs (KPI) par zone arcade

### But
Passer du niveau “parties” à un niveau **pilotage** :
- identifier des zones problématiques,
- mesurer la fréquentation (volume),
- mesurer la diversité (joueurs uniques),
- et estimer la fiabilité (taux d’échec).

### Ressources utilisées
- création d’une variable dérivée  : `is_error`
- `groupby("arcade_zone").agg(...)` pour agréger

### Action
- Création `is_error = 1` si `game_status == "failed"`, sinon 0
- Calcul par zone :
  - `games_count`
  - `unique_players`
  - `error_rate` (moyenne de `is_error`)
  - `avg_pause_s` (pause moyenne)
- Tri sur `error_rate` décroissant

### Résultats et observations 

- Les zones avec les **taux d’erreur les plus élevés** : **C2**, puis **C3**, puis **B2**.
- Les écarts restent **modérés**, et ne semblent pas expliquer directement le volume ou la pause moyenne “au niveau zone”.

### Interprétations
- Une zone “à risque” peut refléter :
  - des machines plus anciennes,
  - un usage plus intensif,
  - ou une maintenance moins fréquente.
- La bonne suite logique est d’analyser **quelles machines** (machine_id) contribuent à l’erreur en C2.

## Visualisation : barplot du taux d’erreur par zone

### But 
Transformer un tableau KPI en **visualisation de comparaison** :
- faciliter la lecture (tri),
- mettre en évidence une zone critique,


### Ressources utilisées
- `seaborn.barplot`
- tri décroissant des zones
- labels (`title`, `xlabel`, `ylabel`)

### Résultat
- Le barplot confirme visuellement que **C2** est la zone la plus critique (taux d’erreur le plus élevé).

### Interprétation & action
- Action recommandée : **drill-down machine** sur C2 (machines présentes, erreurs, codes d’erreur) puis plan de maintenance priorisé.
- Attention méthodo : un taux d’erreur “un peu plus élevé” ne suffit pas toujours → il faut aussi regarder :
  - le **nombre de parties** (stabilité statistique),
  - les **périodes** (effet heure/jour),
  - et la concentration sur une ou deux machines.

  ## Boxplot : performance (durée de jeu) par type de machine

### Objectif
Comparer la **distribution** d’une métrique continue (`game_duration_s`) entre catégories (`machine_type`) :
- médiane (performance “typique”),
- dispersion (variabilité),
- outliers (cas anormaux / anomalies / comportements atypiques).

### Ressources utilisées
- `sns.boxplot`
- une version **avec** outliers
- une version **sans** outliers : `showfliers=False`

### Résultats observés
- Les distributions de durée de jeu semblent **assez similaires** entre types de machines.
- Même sans outliers, on ne voit pas un type “nettement meilleur” en durée de jeu.

### Interprétations
- Le **type de machine** n’est pas un facteur discriminant majeur de la durée (dans ces données).
- Pour expliquer la performance, il faut explorer d’autres variables :
  - `game_mode` (ex. multiball vs speedrun),
  - `pause_s`,
  - `tilt_risk_pct`,
  - `machine_id` (effet machine spécifique),
  - `arcade_zone`,
  - éventuellement le temps (`timestamp`) si certaines périodes changent le comportement.

  ## Score de risque `risk_score`

### But

Mettre en place un **système de priorisation** :
- combiner plusieurs signaux (erreurs, pauses, tilt),
- normaliser les métriques comparables,
- produire un classement “Top 5” actionnable.

### Méthode utilisée
1. Agrégation par `machine_id` :
   - `games_count`
   - `error_rate`
   - `avg_pause_s`
   - `avg_tilt_risk_pct`
2. Filtrage recommandé : **machines avec au moins 30 parties**
3. Normalisation min-max :
   - `pause_norm` dans [0, 1]
   - `tilt_risk_norm` dans [0, 1]
4. Score :
   - `risk_score = 0.5 * error_rate + 0.3 * pause_norm + 0.2 * tilt_risk_norm`

   ### Résultat (exemple de top 5 dans le notebook)
Le classement obtenu met en avant notamment :
- **F10** (fort tilt moyen + error_rate élevé ~8.7%)
- **F07**, **F20**, **F01**, **F04** (selon les poids et la normalisation)

### Interprétations & recommandations (concrètes)
- **Priorité 1 : F10**
  - vérifier capteurs / réglages liés au tilt,
  - audit de maintenance (pannes, calibrage, logs d’erreurs).
- **Surveillance : machines suivantes**
  - mettre en place une alerte si `error_rate` dépasse un seuil,
  - investiguer si la pause moyenne élevée reflète des blocages / latences / comportements joueurs ou un problème de machine.

## Conclusion

1. Une analyse dataviz commence par un **contrôle dataset** (structure, types, volume).
2. Un **nettoyage minimal** évite de biaiser KPI et visualisations (ici ~55 lignes retirées).
3. Les **KPI agrégés** (zone/machine) sont la base du pilotage.
4. Un barplot bien trié rend un diagnostic **immédiat** (C2 zone la plus critique).
5. Les boxplots servent à comparer des **distributions**, pas seulement des moyennes : ici, le `machine_type` n’explique pas fortement la durée.
6. Un **score composite** (normalisé) permet de passer de “constats” à une **liste de priorités actionnables**.

## Pistes d’amélioration (si on prolonge le TP)
- Analyse par `game_mode` (durée, erreur, tilt).
- Drill-down sur C2 : `machine_id` → quelles machines expliquent le taux d’erreur ?
- Corrélations (pause / tilt / vitesse / température) avec `is_error`.
- Analyse temporelle : erreurs par jour/heure (`timestamp`), détection de dérive.
- Visualisation “dashboard” : top machines à risque + évolution temporelle.