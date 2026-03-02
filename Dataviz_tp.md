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