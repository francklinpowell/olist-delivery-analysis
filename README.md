# Olist Delivery Analysis

Analyse des retards de livraison et de leur impact sur la satisfaction client, appliquée au **Brazilian E-Commerce Public Dataset by Olist**.

> Projet réalisé dans le cadre du Certificat Data Analysis — Projet 2.
> Auteur : **Francklin Powell** — Juin 2026

---

## Sommaire

- [Contexte](#contexte)
- [Problématique](#problématique)
- [Aperçu des résultats clés](#aperçu-des-résultats-clés)
- [Architecture du projet](#architecture-du-projet)
- [Structure du dépôt](#structure-du-dépôt)
- [Stack technique](#stack-technique)
- [Installation](#installation)
- [Reproduction du pipeline](#reproduction-du-pipeline)
- [Tableau de bord Power BI](#tableau-de-bord-power-bi)
- [Limites du projet](#limites-du-projet)
- [Pistes d'amélioration](#pistes-damélioration)
- [Licence et source des données](#licence-et-source-des-données)

---

## Contexte

Olist est une marketplace brésilienne qui met en relation des vendeurs et des clients à travers tout le pays. Le Brésil présente une configuration logistique particulièrement intéressante à étudier : un territoire vaste et hétérogène, des infrastructures de transport inégales selon les régions, et une forte concentration économique autour du Sud-Est (São Paulo, Rio de Janeiro).

## Problématique

**Quels sont les facteurs qui expliquent les retards de livraison sur la plateforme Olist, et dans quelle mesure ces retards affectent-ils la satisfaction client ?**

L'analyse cherche à identifier les maillons faibles de la chaîne logistique, qu'ils soient :
- **géographiques** (État du client, distance vendeur-client),
- **temporels** (saisonnalité, jour de commande),
- **liés au produit** (poids, volume, catégorie).

## Aperçu des résultats clés

| Indicateur | Valeur |
|---|---|
| Commandes livrées analysées | 96 454 |
| Délai réel moyen de livraison | 12,1 jours |
| Délai estimé moyen (promis au client) | 23,4 jours |
| Taux de retard global | 7,6 % |
| Score de satisfaction (à l'heure / en avance) | 4,28 / 5 |
| Score de satisfaction (en retard) | 2,44 / 5 |

**Constat principal :** c'est avant tout le franchissement du seuil « à l'heure / en retard », même de quelques jours, qui fait chuter la satisfaction client — bien plus que l'ampleur du retard une fois ce seuil dépassé. Les disparités géographiques (Nord/Nord-Est enclavés vs. Sud-Est industrialisé) et la saisonnalité (pics en février-mars et novembre) sont les deux axes prioritaires d'action identifiés.

Le détail complet des résultats, de la méthodologie et des recommandations est disponible dans le [rapport technique](Rapport_technique_Projet_2.pdf).

## Architecture du projet

Le pipeline suit six grandes étapes :

1. **Compréhension du problème** — traduction de la question métier en indicateurs mesurables.
2. **Chargement des données** — ingestion des 8 fichiers CSV sources via une fonction utilitaire centralisée.
3. **Analyse exploratoire (EDA)** — audit de qualité (dimensions, doublons, valeurs manquantes) sur les fichiers principaux.
4. **Nettoyage et prétraitement** — filtrage des commandes livrées, suppression des doublons et anomalies, imputation des valeurs manquantes.
5. **Construction du schéma en étoile** — restructuration en une table de faits et cinq tables de dimensions, exportées en Excel multi-onglets et en CSV.
6. **Mesures DAX et tableau de bord Power BI** — modélisation relationnelle et construction d'un dashboard interactif à quatre pages.

### Schéma en étoile

| Table | Contenu |
|---|---|
| `fact_orders` | Une ligne par commande : dates clés, indicateurs de délai/retard, fret, prix, satisfaction |
| `dim_customers` | Identifiant client, État et ville de résidence |
| `dim_products` | Identifiant produit, catégorie (traduite en anglais), poids, volume |
| `dim_sellers` | Identifiant vendeur, État et ville d'opération |
| `dim_date` | Calendrier complet (année, mois, trimestre, jour de semaine, week-end) |
| `dim_state` | Référence statique des 27 États brésiliens (nom complet, latitude/longitude) |

## Structure du dépôt

```
olist-delivery-analysis/
├── olist_delivery_analysis.ipynb   # Notebook Jupyter complet : chargement, nettoyage, EDA, KPI, export
├── data/                            # Données sources brutes (8 fichiers CSV Olist/Kaggle)
│   ├── olist_orders_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_geolocation_dataset.csv
│   ├── olist_products_dataset.csv
│   ├── olist_customers_dataset.csv
│   ├── olist_sellers_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   └── olist_product_category_name_translation.csv
├── output_csv/                      # Schéma en étoile exporté (sortie du notebook)
│   ├── fact_orders.csv
│   ├── dim_customers.csv
│   ├── dim_products.csv
│   ├── dim_sellers.csv
│   ├── dim_date.csv
│   └── dim_state.csv
├── olist_powerbi_model.xlsx         # Classeur multi-onglets (alternative à output_csv/ pour l'import Power BI)
├── Rapport PowerBI/                 # Fichier .pbix du tableau de bord à quatre pages
├── Images/                          # Visuels exportés du notebook et du dashboard
│   ├── boxplot_categories.png
│   ├── correlation_matrix.png
│   ├── dashboard_synthese_final.png
│   └── delai_vs_satisfaction.png
├── Logo/                            # Ressources graphiques du dashboard
├── projet2_da.env                   # Variables d'environnement (non versionné, voir Installation)
└── README.md                        # Ce fichier
```

> Le dossier `projet2_da/` (environnement virtuel Python) n'est pas versionné — voir [Installation](#installation) pour le recréer.

## Stack technique

| Catégorie | Outil / Bibliothèque | Usage |
|---|---|---|
| Langage | Python 3.13.1 | Traitement, nettoyage et analyse des données |
| Manipulation de données | Pandas, NumPy | Jointures, agrégations, calculs de KPI |
| Visualisation statique | Matplotlib, Seaborn | Boxplots, matrice de corrélation, dashboard de synthèse |
| Visualisation interactive | Plotly Express / Graph Objects | Histogrammes, cartes choroplèthes, graphiques combinés |
| Environnement de développement | VS Code, Jupyter | Rédaction et exécution du notebook |
| Gestion d'environnement | venv (`projet2_da`), python-dotenv | Isolation des dépendances, gestion sécurisée des identifiants |
| Business Intelligence | Power BI Desktop | Modélisation en étoile, mesures DAX, dashboard 4 pages |
| Gestion de version | Git / GitHub | Versioning et hébergement du dépôt |

## Installation

### Prérequis

- Python 3.13.1 (ou version compatible 3.11+)
- [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (pour ouvrir le fichier `.pbix`, Windows uniquement)
- Un compte Kaggle avec une clé API (pour le téléchargement automatique des données)

### 1. Cloner le dépôt

```bash
git clone https://github.com/francklinpowell/olist-delivery-analysis.git
cd olist-delivery-analysis
```

### 2. Créer et activer l'environnement virtuel

```bash
python -m venv projet2_da

# Windows
projet2_da\Scripts\activate

# macOS / Linux
source projet2_da/bin/activate
```

### 3. Installer les dépendances

```bash
pip install -r requirements.txt
```

> Si `requirements.txt` n'est pas encore présent dans le dépôt, installer manuellement :
> `pip install pandas numpy matplotlib seaborn plotly python-dotenv openpyxl jupyter kaggle`

### 4. Configurer les variables d'environnement

Créer un fichier `projet2_da.env` à la racine du projet. **Ce fichier contient des identifiants et ne doit jamais être versionné** — vérifier qu'il figure bien dans `.gitignore` :

```env
DATA_PATH=./data/
KAGGLE_USERNAME=votre_nom_utilisateur_kaggle
KAGGLE_KEY=votre_cle_api_kaggle
```

> La clé API Kaggle se génère depuis **Compte Kaggle → Settings → API → Create New Token**.

## Reproduction du pipeline

### 1. Récupération des données

Le notebook télécharge automatiquement le dataset via l'API Kaggle si les identifiants sont configurés dans `projet2_da.env`. À défaut, les fichiers CSV peuvent être placés manuellement dans `./data/` depuis la [page Kaggle du dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).

Fichiers requis :
- `olist_orders_dataset.csv`
- `olist_order_items_dataset.csv`
- `olist_geolocation_dataset.csv`
- `olist_products_dataset.csv`
- `olist_customers_dataset.csv`
- `olist_sellers_dataset.csv`
- `olist_order_reviews_dataset.csv`
- `olist_product_category_name_translation.csv`

### 2. Exécution du notebook

```bash
jupyter notebook olist_delivery_analysis.ipynb
```

Exécuter les cellules dans l'ordre (**Run All**). Le notebook réalise successivement :
1. Le chargement et l'audit qualité des données.
2. Le nettoyage et le prétraitement (filtrage, imputation, suppression des doublons et anomalies).
3. Le calcul des KPI globaux et des analyses exploratoires (géographie, catégories, saisonnalité, satisfaction).
4. La construction du schéma en étoile.
5. L'export des tables vers `./output_csv/` (fichiers individuels) et `./olist_powerbi_model.xlsx` (classeur multi-onglets).

### 3. Ouverture du tableau de bord Power BI

1. Ouvrir le fichier `.pbix` situé dans le dossier `Rapport PowerBI/` avec Power BI Desktop.
2. Si nécessaire, mettre à jour la source de données (**Accueil → Transformer les données → Paramètres de la source de données**) pour pointer vers les fichiers exportés à l'étape précédente.
3. Actualiser le modèle (**Accueil → Actualiser**).

## Tableau de bord Power BI

Le dashboard est organisé en **quatre pages interactives**, construites à partir de sept mesures DAX (nombre de commandes, taux de retard, délai moyen, écart d'estimation, fret moyen, etc.) :

1. **Vue d'ensemble** — KPI globaux et évolution mensuelle des commandes et de la satisfaction.
2. **Analyse géographique** — cartographie des délais et taux de retard par État, comparaison vendeur/client.
3. **Produits & Transport** — délai et fret par quartile de poids, taux de retard par catégorie de produit.
4. **Recommandations** — saisonnalité des retards et synthèse des recommandations stratégiques.

## Limites du projet

- **Biais géographique** : São Paulo concentre une part très importante du volume de commandes ; les statistiques sur les petits États du Nord reposent sur des échantillons plus restreints.
- **Couverture temporelle incomplète** : données 2016 limitées, et fenêtre de collecte s'arrêtant en septembre-octobre 2018.
- **Biais de sélection potentiel sur la satisfaction** : les clients insatisfaits sont généralement plus enclins à laisser un avis.
- **Absence d'identifiant transporteur** : l'État du vendeur comparé à l'État du client est utilisé comme proxy indirect.
- **Pas de lien de causalité strict** établi entre les corrélations observées (retard/satisfaction, éloignement/délai) — d'autres variables non observées (densité de population, infrastructures, climat) pourraient intervenir.

Le détail complet des biais et limites méthodologiques est disponible en section 7 du [rapport technique](Rapport_technique_Projet_2.pdf).

## Pistes d'amélioration

- Enrichir le dataset avec des données externes (météo, jours fériés, infrastructures routières).
- Obtenir un identifiant transporteur réel pour comparer la performance logistique par prestataire.
- Automatiser le pipeline de bout en bout (téléchargement → nettoyage → export → actualisation Power BI).
- Déployer un modèle prédictif du risque de retard au moment de la commande.
- Mettre le tableau de bord en production avec actualisation automatique et alertes sur les seuils critiques.

## Licence et source des données

Le jeu de données **Brazilian E-Commerce Public Dataset by Olist** est disponible publiquement sur [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) et couvre les commandes passées entre 2016 et 2018.

---

📄 Rapport technique complet : [`Rapport_technique_Projet_2.pdf`](Rapport_technique_Projet_2.pdf)
