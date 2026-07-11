# Pandas

## Introduction

### Qu'est-ce que Pandas ?
**Pandas** est la bibliothèque open-source de référence en Python pour la manipulation, le nettoyage et l'analyse de données structurées. Développée initialement par Wes McKinney en 2008, elle repose sur NumPy (ce qui lui confère une grande vitesse de calcul en C) et s'intègre parfaitement avec tout l'écosystème scientifique et Machine Learning de Python (Scikit-Learn, Matplotlib, Seaborn, SciPy).

### Pourquoi Pandas est important en Data Science et Machine Learning ?
En Machine Learning, l'adage *"Garbage in, garbage out"* (si les données d'entrée sont mauvaises, les prédictions le seront aussi) est fondamental. Plus de **80% du temps** d'un projet de ML/Data Science est consacré à la préparation des données. Pandas fournit des structures de données intuitives et rapides pour :
* Manipuler des données hétérogènes (numériques, textuelles, dates, booléennes).
* Gérer facilement les valeurs manquantes (`NaN` / `None`).
* Restructurer, filtrer et agréger les données.
* Effectuer des analyses exploratoires rapides (EDA - Exploratory Data Analysis).

### Rôle de Pandas dans un pipeline ML
Dans un pipeline de Machine Learning type, Pandas se situe juste après l'acquisition des données et juste avant la modélisation mathématique :

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│ 1. Sources Brutes│ ──> │ 2. Pandas (Clean)│ ──> │ 3. Feature Eng. (PD) │ ──> │ 4. Scikit-Learn (ML)│
│ (CSV, SQL, JSON) │     │ (NaN, Types, Dup)│     │ (Encoding, Scaling)  │     │ (fit / predict)     │
└──────────────────┘     └──────────────────┘     └──────────────────────┘     └─────────────────────┘
```

### Comparaison rapide avec les structures Python natives

| Caractéristique | Liste Python (`list`) | Dictionnaire Python (`dict`) | DataFrame / Series Pandas |
| :--- | :--- | :--- | :--- |
| **Type de données** | Homogène ou Hétérogène | Clé-Valeur | Homogène par colonne (performances C) |
| **Vitesse (Calcul)** | Lent (boucles Python) | Moyen (recherche rapide) | Très rapide (vectorisation vectorielle NumPy) |
| **Indexation** | Positionnelle uniquement (0, 1...) | Par clé uniquement | Positionnelle ET par Label personnalisé |
| **Valeurs Manquantes**| Gestion manuelle fastidieuse | `None` manuel | Alignement automatique et fonctions intégrées |
| **Alignement** | Aucun alignement automatique | Aucun alignement automatique | Alignement automatique lors des calculs |

---

## Installation

Pour installer Pandas dans votre environnement Python :

```bash
pip install pandas
```

*Note : Pour des fonctionnalités étendues (gestion des fichiers Excel, Parquet, SQL), vous pouvez installer les dépendances associées (`pip install openpyxl pyarrow sqlalchemy`).*

### Importation de Pandas et vérification de la version

```python
import pandas as pd
import numpy as np

# Vérifier la version installée
print(f"Version de Pandas : {pd.__version__}")
print(f"Version de NumPy : {np.__version__}")
```

---

## Pandas Data Structures

Pandas possède deux structures de données fondamentales : la **Series** (unidimensionnelle) et le **DataFrame** (bidimensionnel).

---

### Series

Une **Series** est un tableau unidimensionnel étiqueté capable de contenir n'importe quel type de données. Elle se compose de deux éléments clés :
1. Les **données** (Values).
2. L'**index** (les étiquettes/labels des lignes).

```
Structure d'une Series :
         Index    Values
        ┌─────┐  ┌────────┐
      0 │  a  │  │  10.5  │
      1 │  b  │  │  20.3  │
      2 │  c  │  │  15.8  │
        └─────┘  └────────┘
```

#### Création d'une Series

```python
# 1. Création à partir d'une liste (index généré automatiquement)
data_list = [10, 20, 30, 40]
s1 = pd.Series(data_list)
print("Series depuis liste :\n", s1)

# 2. Création avec index personnalisé
s2 = pd.Series(data_list, index=['A', 'B', 'C', 'D'])
print("\nSeries avec index personnalisé :\n", s2)

# 3. Création depuis un dictionnaire (les clés deviennent l'index)
data_dict = {'Paris': 2148000, 'Lyon': 515000, 'Marseille': 861000}
s3 = pd.Series(data_dict)
print("\nSeries depuis dictionnaire :\n", s3)
```

#### Accès aux valeurs (Indexing & Slicing)

```python
# Accès par label (index explicite)
print("Population de Paris :", s3['Paris'])

# Accès par position (index implicite)
print("Deuxième élément de s2 :", s2[1])

# Slicing
print("\nSlicing de s2 (de A à C) :\n", s2['A':'C'])
```

#### Opérations principales sur les Series

```python
# Opérations mathématiques vectorisées (sans boucle)
s_double = s2 * 2
print("Series multipliée par 2 :\n", s_double)

# Opérations statistiques de base
print("\nMoyenne de s2 :", s2.mean())
print("Valeur maximale de s2 :", s2.max())
```

---

### DataFrame

Un **DataFrame** est une structure de données bidimensionnelle (comme un tableau SQL ou une feuille Excel). Il possède à la fois un index de lignes et un index de colonnes.

```
Structure d'un DataFrame :
                     Columns (Column Index)
                   ┌─────────┬─────────┐
                   │  Nom    │  Âge    │
                 ┌─┼─────────┼─────────┤
      Row Index  │0│  Alice  │   25    │
                 ├─┼─────────┼─────────┤
                 │1│  Bob    │   30    │
                 └─┴─────────┴─────────┘
```

#### Création d'un DataFrame depuis différentes sources

```python
# 1. Depuis un dictionnaire de listes
data_dict = {
    'Nom': ['Alice', 'Bob', 'Charlie', 'David'],
    'Age': [25, 30, 35, 40],
    'Ville': ['Paris', 'Lyon', 'Marseille', 'Nice']
}
df_dict = pd.DataFrame(data_dict)
print("DataFrame depuis dictionnaire :\n", df_dict)

# 2. Depuis une liste de listes (avec spécification des colonnes)
data_list = [
    ['Alice', 25, 'Paris'],
    ['Bob', 30, 'Lyon'],
    ['Charlie', 35, 'Marseille']
]
df_list = pd.DataFrame(data_list, columns=['Nom', 'Age', 'Ville'])
print("\nDataFrame depuis liste :\n", df_list)

# 3. Depuis un Array NumPy
np_array = np.random.randint(10, 50, size=(3, 3))
df_np = pd.DataFrame(np_array, columns=['Col_A', 'Col_B', 'Col_C'])
print("\nDataFrame depuis NumPy array :\n", df_np)
```

---

## Loading Data

Pandas permet d'importer des jeux de données depuis de multiples formats. Voici les fonctions principales et leurs paramètres clés.

| Format | Fonction | Paramètres clés fréquemment utilisés |
| :--- | :--- | :--- |
| **CSV** | `pd.read_csv()` | `sep` (séparateur), `header` (ligne d'en-tête), `index_col` (colonne d'index), `usecols` (sélection de colonnes), `nrows` (nombre de lignes à charger) |
| **Excel** | `pd.read_excel()`| `sheet_name` (feuille de calcul), `header` |
| **JSON** | `pd.read_json()` | `orient` (structure du JSON : 'records', 'index', etc.) |
| **SQL** | `pd.read_sql()` | `sql` (requête SQL ou nom de table), `con` (connexion SQLAlchemy ou SQLite) |
| **Parquet**| `pd.read_parquet()`| `engine` (pyarrow ou fastparquet), `columns` |

### Exemples d'importation de données

```python
# 1. Charger un fichier CSV avec délimiteur personnalisé (ex: point-virgule)
# df = pd.read_csv('data.csv', sep=';', encoding='utf-8')

# 2. Charger un fichier Excel avec feuille spécifique
# df = pd.read_excel('report.xlsx', sheet_name='Q4_2026')

# 3. Charger un fichier JSON
# df = pd.read_json('users.json', orient='records')

# 4. Lire depuis une base de données SQL
# from sqlalchemy import create_engine
# engine = create_engine('sqlite:///database.db')
# df = pd.read_sql('SELECT * FROM client WHERE age > 30', con=engine)

# 5. Lire un fichier compressé Parquet (optimisé pour le Big Data)
# df = pd.read_parquet('data.parquet')
```

---

## Exploring Data

Dès qu'un jeu de données est importé, il est indispensable de l'explorer. Pandas fournit des méthodes clés pour obtenir un aperçu rapide.

```python
# Création d'un dataset d'exemple pour l'exploration
data = {
    'Nom': ['Alice', 'Bob', 'Charlie', 'David', 'Eva', None],
    'Age': [25, 30, 35, 40, np.nan, 29],
    'Profession': ['Data Scientist', 'ML Engineer', 'Data Analyst', 'DevOps', 'ML Engineer', 'Data Scientist'],
    'Salaire': [75000, 80000, 62000, 68000, 82000, 71000]
}
df = pd.DataFrame(data)

# 1. head(n) et tail(n) : Afficher les n premières / dernières lignes
print("Les 3 premières lignes :")
print(df.head(3))

# 2. info() : Structure du DataFrame, types de données et valeurs non-nulles
print("\nStructure et types :")
df.info()

# 3. describe() : Résumé statistique des colonnes numériques (et textuelles avec include='all')
print("\nRésumé statistique :")
print(df.describe())

# 4. shape, columns, dtypes : Attributs de base
print("\nDimensions du DataFrame (lignes, colonnes) :", df.shape)
print("Noms des colonnes :", df.columns.tolist())
print("Types des colonnes :\n", df.dtypes)

# 5. value_counts() : Fréquence des modalités d'une variable catégorielle
print("\nNombre d'occurrences par Profession :")
print(df['Profession'].value_counts())
```

---

## Data Selection and Indexing

Sélectionner des sous-ensembles de données est l'une des tâches les plus fréquentes. Pandas offre plusieurs syntaxes puissantes.

### Sélection de colonnes et de lignes de base

```python
# Sélection d'une colonne (retourne une Series)
ages = df['Age']

# Sélection de plusieurs colonnes (retourne un DataFrame)
sub_df = df[['Nom', 'Salaire']]

# Sélection de lignes par indexation classique (Slicing de base)
print(df[0:2]) # Deux premières lignes
```

### Sélection avec `.loc` (basée sur les labels) et `.iloc` (basée sur les positions)

```python
# Syntaxe générale : df.loc[index_lignes, index_colonnes]

# Sélectionner la ligne d'index 2 pour la colonne 'Nom'
print(df.loc[2, 'Nom']) # Charlie

# Sélectionner les lignes de 0 à 2 (inclus) pour 'Nom' et 'Profession'
print(df.loc[0:2, ['Nom', 'Profession']])

# Sélectionner par position d'index avec .iloc (exclut la borne supérieure de fin)
# Lignes 0 et 1, colonnes 0 et 2
print(df.iloc[0:2, [0, 2]])
```

### Filtrage Conditionnel (Boolean Indexing)

```python
# Filtrer les individus ayant un salaire supérieur à 70000
high_salary = df[df['Salaire'] > 70000]
print("Salaires > 70000 :\n", high_salary)

# Filtrage multi-conditions (utilisez obligatoirement les opérateurs binaires &, |, ~ et entourez de parenthèses)
# Condition : Age > 28 ET Profession == 'Data Scientist'
cond = (df['Age'] > 28) & (df['Profession'] == 'Data Scientist')
filtered_df = df[cond]
print("\nFiltrage multi-conditions (Age > 28 AND Data Scientist) :\n", filtered_df)

# Condition : Profession est soit 'ML Engineer', soit 'DevOps'
filtered_prof = df[df['Profession'].isin(['ML Engineer', 'DevOps'])]
print("\nFiltrage par liste (.isin) :\n", filtered_prof)
```

---

## Data Cleaning

Les données réelles contiennent souvent du bruit, des doublons, des types incohérents et des valeurs manquantes.

### Missing Values (Valeurs Manquantes)

```python
# Détecter les valeurs manquantes (isnull ou isna)
print("Valeurs manquantes par colonne :\n", df.isnull().sum())

# Option 1 : Supprimer les lignes contenant au moins une valeur manquante
df_dropped = df.dropna()

# Option 2 : Supprimer uniquement les lignes où 'Nom' est manquant
df_dropped_subset = df.dropna(subset=['Nom'])

# Option 3 : Imputer (remplir) les valeurs manquantes par une valeur fixe ou statistique
df_filled = df.copy()
df_filled['Age'] = df_filled['Age'].fillna(df_filled['Age'].mean()) # imputation par la moyenne
print("\nDataFrame après imputation de l'âge :\n", df_filled)
```

### Duplicate Data (Doublons)

```python
# Repérer les lignes en doublon
print("Doublons détectés :", df.duplicated().sum())

# Supprimer les doublons (conserve la 1ère occurrence par défaut)
df_clean = df.drop_duplicates()
```

### Data Types (Conversions)

```python
# Convertir une colonne en type spécifique (ex: float vers int/Int64 pour tolérer les NaN)
df_filled['Age'] = df_filled['Age'].astype('int64')

# Convertir des colonnes textuelles en dates
date_df = pd.DataFrame({'date_str': ['2026-01-01', '2026-06-15', '2026-12-31']})
date_df['date_parsed'] = pd.to_datetime(date_df['date_str'])
print("\nTypes de colonnes après conversion date :\n", date_df.dtypes)
```

### String Processing (Traitement de chaînes de caractères)

Toutes les méthodes de chaînes Python standard sont accessibles via l'accesseur `.str` sur une Series.

```python
# Convertir en majuscules
df_filled['Nom_Upper'] = df_filled['Nom'].str.upper()

# Remplacer une chaîne
df_filled['Profession_Clean'] = df_filled['Profession'].str.replace('ML', 'Machine Learning')

# Extraire les premières lettres d'une chaîne
df_filled['Initiale'] = df_filled['Nom'].str.slice(0, 1)
print(df_filled[['Nom', 'Nom_Upper', 'Profession_Clean']])
```

---

## Data Transformation

Appliquer des fonctions personnalisées ou vectorisées sur un DataFrame est essentiel pour le Feature Engineering.

```python
df_trans = df_filled.copy()

# 1. .map() : Applique une fonction ou un dictionnaire élément par élément sur une Series
prof_mapping = {'Data Scientist': 'Data', 'ML Engineer': 'ML', 'Data Analyst': 'Data', 'DevOps': 'Ops'}
df_trans['Prof_Abrege'] = df_trans['Profession'].map(prof_mapping)

# 2. .apply() : Applique une fonction sur l'axe des lignes (axis=0) ou colonnes (axis=1)
# Exemple : Calculer le salaire net estimé (salaire brut * 0.78) via lambda fonction
df_trans['Salaire_Net'] = df_trans['Salaire'].apply(lambda x: x * 0.78)

# 3. .applymap() (renommé .map() à partir de Pandas 2.1+) : Applique une fonction à tout le DataFrame élément par élément
# Exemple : Convertir tous les éléments textuels en chaînes en minuscules
df_str_only = df_trans[['Nom', 'Profession']].astype(str).map(str.lower)
print(df_str_only.head(2))
```

### Comparaison des approches

| Méthode | Cible | Utilisation classique | Performances |
| :--- | :--- | :--- | :--- |
| **Vectorisation** | Colonne entière | Opérations mathématiques de base (`df['A'] * 2`) | **Ultra-rapide** (optimisé C) |
| **`.apply(lambda)`**| Lignes / Colonnes | Fonctions complexes, logiques conditionnelles | **Moyenne** (boucle sous-jacente) |
| **`.map()`** | Une Series | Mapping par dictionnaire ou traduction | **Rapide** (spécifique Series) |

---

## Sorting and Ranking

```python
# Trier par ordre croissant du Salaire
df_sorted = df_filled.sort_values(by='Salaire', ascending=True)

# Trier par plusieurs colonnes (Profession, puis Salaire décroissant)
df_multi_sorted = df_filled.sort_values(by=['Profession', 'Salaire'], ascending=[True, False])

# Classer (Ranking) : Donner un rang à chaque ligne selon une colonne
df_filled['Rang_Salaire'] = df_filled['Salaire'].rank(ascending=False)
print(df_filled[['Nom', 'Salaire', 'Rang_Salaire']])
```

---

## GroupBy Operations

La méthode `.groupby()` permet d'analyser les données selon le paradigme **Split-Apply-Combine** :
1. **Split** : Diviser les données en groupes selon des critères.
2. **Apply** : Appliquer une fonction statistique ou personnalisée sur chaque groupe.
3. **Combine** : Fusionner les résultats dans une structure unique.

```
Paradigme GroupBy :
   Données de base        1. Split           2. Apply (Mean)        3. Combine
   ┌───┬───────┐        ┌─────────────┐        ┌──────────┐        ┌───┬───────┐
   │Cat│ Valeur│        │ A: [10, 20] │ ─────> │ A: 15.0  │ ─────> │Cat│ Moy   │
   ├───┼───────┤        ├─────────────┤        ├──────────┤        ├───┼───────┤
   │ A │  10   │ ─────> │ B: [30, 40] │ ─────> │ B: 35.0  │        │ A │ 15.0  │
   │ B │  30   │        └─────────────┘        └──────────┘        │ B │ 35.0  │
   │ A │  20   │                                                   └───┴───────┘
   │ B │  40   │
   └───┴───────┘
```

### Exemples d'agrégation simples et multiples

```python
# 1. Agrégation simple : Moyenne de salaire par Profession
print("Salaire moyen par profession :")
print(df_filled.groupby('Profession')['Salaire'].mean())

# 2. Agrégations multiples : Calculer la moyenne, l'écart-type et le max de Salaire par Profession
print("\nStatistiques multiples par profession :")
stats = df_filled.groupby('Profession')['Salaire'].agg(['mean', 'std', 'max', 'count'])
print(stats)

# 3. Agrégations nommées (Named Aggregation) pour renommer directement les colonnes résultantes
print("\nAgrégations nommées :")
named_agg = df_filled.groupby('Profession').agg(
    salaire_moyen=('Salaire', 'mean'),
    age_max=('Age', 'max'),
    nombre_employes=('Nom', 'count')
)
print(named_agg)
```

---

## Merge, Join and Concatenation

Associer plusieurs sources de données est indispensable. Pandas fournit 3 outils complémentaires.

### 1. `merge()`
Equivalent du `JOIN` en SQL. Utilisé pour joindre des DataFrames via des clés de correspondance.

```python
df1 = pd.DataFrame({'Emp_ID': [1, 2, 3], 'Nom': ['Alice', 'Bob', 'Charlie']})
df2 = pd.DataFrame({'Emp_ID': [2, 3, 4], 'Projet': ['Alpha', 'Beta', 'Gamma']})

# Inner Join (Intersection - clé présente dans les deux)
print("Inner Join :\n", pd.merge(df1, df2, on='Emp_ID', how='inner'))

# Left Join (Conserve tout df1, remplit df2 par des NaN si non trouvé)
print("\nLeft Join :\n", pd.merge(df1, df2, on='Emp_ID', how='left'))

# Outer Join (Union complète - conserve toutes les données des deux côtés)
print("\nOuter Join :\n", pd.merge(df1, df2, on='Emp_ID', how='outer'))
```

### 2. `concat()`
Permet d'empiler des DataFrames verticalement (axe 0) ou horizontalement (axe 1).

```python
# Concaténation verticale (ajouter des lignes)
df_a = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df_b = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})
df_vertical = pd.concat([df_a, df_b], ignore_index=True)
print("\nConcaténation verticale :\n", df_vertical)
```

### 3. `join()`
Similaire à `merge()`, mais optimisé pour une jointure basée sur les index.

---

## Working with Dates and Time Series

Pandas est particulièrement puissant pour traiter les séries temporelles (Time Series), courantes en finance ou IoT.

```python
# Création d'une série temporelle
dates = pd.date_range(start='2026-01-01', end='2026-01-05', freq='D')
ts = pd.DataFrame({'Ventes': [100, 150, 120, 200, 180]}, index=dates)
print("Série temporelle :\n", ts)

# Extraction d'éléments de la date (via .index.year, .index.day_name(), etc.)
ts['Année'] = ts.index.year
ts['Mois'] = ts.index.month
ts['Jour'] = ts.index.day_name()
print("\nDates enrichies :\n", ts)
```

### Resampling et Rolling (Fenêtres glissantes)

```python
# 1. Resampling : Modifier la fréquence temporelle (ex: passer de quotidien à hebdomadaire 'W')
# ts.resample('W').mean()

# 2. Rolling (Fenêtres glissantes) : Calculer une moyenne mobile sur 3 jours
ts['Moyenne_Mobile_3J'] = ts['Ventes'].rolling(window=3).mean()
print("\nAvec Moyenne Mobile :\n", ts)
```

---

## Data Visualization with Pandas

Pandas s'interface directement avec **Matplotlib** sous le capot pour permettre de tracer des graphiques en une seule ligne de code.

```python
# Pour visualiser, importez matplotlib
import matplotlib.pyplot as plt

# Exemple de données
df_vis = pd.DataFrame({
    'Mois': ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
    'CA': [12000, 15000, 14000, 18000, 22000],
    'Depenses': [8000, 9500, 11000, 12000, 13000]
})

# 1. Line Chart
df_vis.plot(x='Mois', y=['CA', 'Depenses'], kind='line', marker='o')
plt.title("Évolution du Chiffre d'Affaires")
plt.ylabel("Montant (€)")
# plt.show() # Décommenter pour afficher le graphique

# 2. Bar Chart
df_vis.plot(x='Mois', y='CA', kind='bar', color='skyblue')
# plt.show()

# 3. Histogramme (distribution de données)
# df_vis['CA'].plot(kind='hist', bins=5)

# 4. Scatter Plot (Corrélation)
df_vis.plot(kind='scatter', x='CA', y='Depenses', color='red')
```

---

## Pandas for Machine Learning

En Machine Learning, Pandas sert de passerelle entre les données brutes et les tenseurs NumPy passés aux modèles.

```
┌──────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
│  Dataset Pandas  │ ───> │ Séparation Features/y  │ ───> │ Encodage Catégoriel    │ ───> │  Conversion NumPy      │
│  (DataFrame)     │      │ X = df[[c1, c2]], y    │      │ (pd.get_dummies)       │      │  .values (Scikit-Learn)│
└──────────────────┘      └────────────────────────┘      └────────────────────────┘      └────────────────────────┘
```

### Exemple de pipeline de préparation

```python
# Dataset ML brut d'exemple
data_ml = pd.DataFrame({
    'Taille_m2': [50, 75, 120, 45, 90],
    'Nb_Chambres': [2, 3, 4, 1, 3],
    'Quartier': ['Centre', 'Banlieue', 'Centre', 'Nord', 'Banlieue'],
    'Prix': [250000, 310000, 520000, 190000, 380000]
})

# 1. Séparation Variables Explicatives (X) / Variable Cible (y)
X = data_ml.drop(columns='Prix')
y = data_ml['Prix']

# 2. Encodage des variables catégorielles (One-Hot Encoding)
X_encoded = pd.get_dummies(X, columns=['Quartier'], drop_first=True)

# 3. Conversion en Array NumPy pour Scikit-Learn (Optionnel, Sklearn accepte les DataFrames)
X_np = X_encoded.values
y_np = y.values

print("X encodé prêt pour le ML :\n", X_encoded)
```

---

## Feature Engineering with Pandas

Le Feature Engineering consiste à créer de nouvelles variables explicatives à partir des données brutes pour améliorer les performances d'un modèle prédictif.

```python
df_fe = data_ml.copy()

# 1. Création d'une variable d'interaction (ratio)
df_fe['Prix_m2_Moyen'] = df_fe['Prix'] / df_fe['Taille_m2']

# 2. Binarisation / Seuil
# Déterminer si un bien est grand (Taille > 80m2)
df_fe['Grand_Logement'] = (df_fe['Taille_m2'] > 80).astype(int)

# 3. Binning (Discrétisation de variables numériques en intervalles)
# Créer 3 catégories de prix : Bas, Moyen, Haut
df_fe['Categorie_Prix'] = pd.cut(df_fe['Prix'], bins=3, labels=['Bas', 'Moyen', 'Haut'])

print("Nouveaux Features :\n", df_fe)
```

---

## Performance Optimization

Lorsque les DataFrames atteignent plusieurs gigaoctets, le code standard peut devenir trop lent. Voici les clés pour optimiser les performances.

### 1. Bannir les boucles Python (`for index, row in df.iterrows()`)
Parcourir un DataFrame ligne par ligne avec une boucle Python est l'antipattern absolu en Pandas. La vectorisation applique les opérations à bas niveau via C.

*Exemple de comparaison de vitesse :*
```python
# A ÉVITER : Lent et inefficace
# for i in range(len(df)):
#     df.loc[i, 'Nouveau'] = df.loc[i, 'Salaire'] * 1.10

# À PRIVILÉGIER : Vectorisé, immédiat
# df['Nouveau'] = df['Salaire'] * 1.10
```

### 2. Optimiser les dtypes (Types de données)
Par défaut, Pandas alloue beaucoup de mémoire (ex: `int64` pour les petits entiers, ou `object` pour les chaînes).
* Convertir les colonnes textuelles répétitives en type `category`.
* Utiliser des entiers plus petits (`int8`, `int16`) lorsque les plages de valeurs le permettent.

```python
# Réduction de la mémoire en changeant de type
df_opt = pd.DataFrame({'Genre': ['Homme', 'Femme', 'Homme'] * 1000})
print("Mémoire initiale :", df_opt.memory_usage(deep=True).sum(), "octets")

df_opt['Genre'] = df_opt['Genre'].astype('category')
print("Mémoire optimisée (category) :", df_opt.memory_usage(deep=True).sum(), "octets")
```

---

## Large Dataset Handling

Lorsque le volume de données dépasse la RAM physique disponible :

### 1. Lecture par blocs (Chunk Processing)
Le paramètre `chunksize` permet de charger le fichier CSV petit à petit (par exemple 10 000 lignes à la fois).

```python
# Traiter un fichier volumineux par morceaux
# chunk_size = 10000
# for chunk in pd.read_csv('giant_dataset.csv', chunksize=chunk_size):
#     # Effectuer des agrégations intermédiaires sur chaque bloc
#     process(chunk)
```

### 2. Alternatives à Pandas pour le Big Data
Lorsque Pandas montre ses limites (limité au traitement monocœur en mémoire RAM) :
* **Polars** : Bibliothèque ultra-rapide écrite en Rust, multithreadée, utilisant une évaluation paresseuse (Lazy Evaluation).
* **Dask** : Permet de paralléliser les calculs Pandas sur plusieurs cœurs ou sur un cluster de machines.
* **Apache Spark / PySpark** : Pour le traitement de données massivement distribuées.

---

## Common Errors and Debugging

### 1. `SettingWithCopyWarning`
* **Pourquoi ?** Vous tentez de modifier une portion de DataFrame obtenue par filtrage (qui peut être une simple vue temporaire au lieu d'une copie autonome).
* **Mauvais code :**
  ```python
  sub_df = df[df['Age'] > 30]
  sub_df['Statut'] = 'Senior'  # Déclenche le Warning
  ```
* **Solution :** Utiliser explicitement `.copy()` pour créer un nouvel objet indépendant, ou utiliser `.loc` directement sur le DataFrame d'origine.
  ```python
  sub_df = df[df['Age'] > 30].copy()
  sub_df['Statut'] = 'Senior'  # OK
  ```

### 2. `KeyError`
* **Pourquoi ?** Vous essayez d'accéder à une colonne ou un index qui n'existe pas.
* **Solution :** Vérifier l'orthographe exacte via `df.columns` ou les espaces résiduels via `df.columns = df.columns.str.strip()`.

### 3. `MemoryError`
* **Pourquoi ?** Le jeu de données dépasse la taille de la RAM.
* **Solution :** Utiliser `usecols` dans `read_csv` pour ne charger que les colonnes nécessaires, modifier les `dtypes` ou utiliser **Polars**.

---

## Best Practices

1. **Éviter les modifications sur place (`inplace=True`)** : Ce paramètre obsolète ou déprécié dans de nombreuses fonctions n'améliore pas les performances et empêche le chaînage d'opérations (method chaining). Privilégiez l'affectation directe : `df = df.dropna()`.
2. **Utiliser le chaînage de méthodes (Method Chaining)** : Rendre le code fluide et lisible.
   ```python
   # Code chaîné, propre et lisible :
   df_clean = (df.dropna(subset=['Salaire'])
                 .assign(Salaire_Mensuel=lambda x: x['Salaire'] / 12)
                 .query("Age > 25")
                 .sort_values(by='Salaire_Mensuel'))
   ```
3. **Valider les types et données après chaque étape** : Utiliser `df.shape` et `df.dtypes` régulièrement pour s'assurer que les transformations se comportent comme prévu.

---

## Complete Mini Project

Voici un projet complet et autonome simulant un cycle complet : Chargement d'un fichier brut -> Nettoyage -> Feature Engineering -> Préparation pour le Machine Learning.

```python
import pandas as pd
import numpy as np

# --- ÉTAPE 1 : Génération et sauvegarde d'un jeu de données brut factice ---
raw_data = """id;customer_name;age;yearly_income;signup_date;churned
101;Alice Smith;34;75000;2024-03-12;no
102;Bob Jones;45;120000;2023-11-05;yes
103;Charlie Brown;;62000;2025-01-20;no
104;David Miller;29;85000;2024-07-30;no
105;Eva Davis;38;;2024-05-15;no
106;Bob Jones;45;120000;2023-11-05;yes
"""
with open('customers_raw.csv', 'w', encoding='utf-8') as f:
    f.write(raw_data)

print("--- Fichier brut créé ---")

# --- ÉTAPE 2 : Chargement des données ---
df = pd.read_csv('customers_raw.csv', sep=';')
print("\n[Données initiales] :\n", df)

# --- ÉTAPE 3 : Nettoyage des données ---
# 1. Supprimer les doublons
df = df.drop_duplicates()

# 2. Corriger les valeurs manquantes
# Imputer l'âge par la médiane
df['age'] = df['age'].fillna(df['age'].median())
# Imputer le revenu annuel (yearly_income) par la moyenne
df['yearly_income'] = df['yearly_income'].fillna(df['yearly_income'].mean())

# 3. Convertir les types
df['age'] = df['age'].astype(int)
df['signup_date'] = pd.to_datetime(df['signup_date'])

# --- ÉTAPE 4 : Feature Engineering ---
# 1. Calculer l'ancienneté du client en jours par rapport à une date fixe de référence
ref_date = pd.to_datetime('2026-07-01')
df['membership_days'] = (ref_date - df['signup_date']).dt.days

# 2. Créer une classe d'âge
df['age_group'] = pd.cut(df['age'], bins=[0, 30, 45, 100], labels=['Jeune', 'Adulte', 'Senior'])

# 3. Encoder la variable binaire cible 'churned' en numérique (yes -> 1, no -> 0)
df['churned'] = df['churned'].map({'yes': 1, 'no': 0})

print("\n[Données après nettoyage et Feature Engineering] :\n", df)

# --- ÉTAPE 5 : Préparation finale pour le Machine Learning ---
# Sélection des features d'apprentissage et de la cible
features = df[['age', 'yearly_income', 'membership_days', 'age_group']]
target = df['churned']

# Encodage One-Hot des variables catégorielles (age_group)
features_processed = pd.get_dummies(features, columns=['age_group'], drop_first=True)

# Export final
features_processed.to_csv('features_ready.csv', index=False)
print("\n[Dataset final prêt exporté avec succès] :\n", features_processed)
```

---

## Pandas Cheat Sheet

Voici les commandes incontournables à retenir :

| Action | Commande |
| :--- | :--- |
| **Importer** | `import pandas as pd` |
| **Lire un CSV** | `df = pd.read_csv('file.csv')` |
| **Écrire un CSV** | `df.to_csv('file.csv', index=False)` |
| **Aperçu** | `df.head()` / `df.info()` / `df.describe()` |
| **Sélectionner colonne(s)** | `df['nom_colonne']` / `df[['col1', 'col2']]` |
| **Sélectionner par label** | `df.loc[ligne_label, colonne_label]` |
| **Sélectionner par position**| `df.iloc[ligne_pos, colonne_pos]` |
| **Filtrer** | `df[df['colonne'] > valeur]` |
| **Nettoyer les manquants** | `df.dropna()` / `df.fillna(valeur)` |
| **Grouper** | `df.groupby('colonne_categorie')['valeurs'].mean()` |
| **Fusionner (Join)** | `pd.merge(df1, df2, on='cle_commune')` |
| **Calculer une moyenne** | `df['colonne'].mean()` |

---

## Conclusion

Pandas est la colonne vertébrale de l'écosystème Python pour la science des données. Sa maîtrise permet de manipuler les jeux de données les plus complexes avec un minimum de lignes de code.

* **Quand utiliser Pandas** : Pour tout projet impliquant des données tabulaires de taille petite à moyenne (< 10 Go), l'analyse exploratoire et les pipelines de Machine Learning classiques.
* **Quand s'orienter vers des alternatives** : Dès que les données dépassent les capacités de votre mémoire vive (RAM) ou que les temps de calcul deviennent bloquants (choisissez alors **Polars** ou **Dask/Spark**).
