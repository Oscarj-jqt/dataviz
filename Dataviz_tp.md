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