# 📊 Data Warehouse - Secteur Automobile (DWH_E5_projet_AUTO)

## 🎯 Description du Projet

Projet de création et d'alimentation d'un **Data Warehouse** dédié au secteur automobile dans la région Hauts-de-France. Le DWH centralise des données provenant de sources multiples (base opérationnelle Carter Cash, Azure Data Lake, fichiers CSV) pour permettre des analyses décisionnelles sur trois axes métier :

- **Pneumatiques** : Prix, caractéristiques techniques, compatibilité véhicules
- **Mobilité électrique** : Bornes de recharge IRVE (Infrastructure de Recharge de Véhicules Électriques)
- **Tissu économique** : Entreprises du secteur automobile (codes NAF 45.x)

## 🏗️ Architecture du Data Warehouse

### **Tables de Dimensions (10)**
- `DIM_TEMPS` : Dimension temporelle (2020-2030)
- `DIM_GEOGRAPHIE` : Communes des Hauts-de-France (59, 62, 80, 60, 02)
- `DIM_ENTREPRISE` : Établissements secteur automobile (SCD Type 2)
- `DIM_ACTIVITE_NAF` : Nomenclature des activités (NAF Rev.2)
- `DIM_BORNE_RECHARGE` : Bornes de recharge électrique
- `DIM_PRODUIT_PNEU`, `DIM_CARACTERISTIQUES_PNEU`, `DIM_DIMENSIONS_PNEU` : Catalogue pneumatiques
- `DIM_VEHICULE` : Modèles de véhicules compatibles
- `DIM_USER_API` : Utilisateurs de l'API Carter Cash

### **Tables de Faits (3)**
- `FAIT_PRIX_PNEU` : Évolution des prix des pneumatiques
- `FAIT_DISPONIBILITE_BORNE` : Disponibilité et capacité des bornes de recharge
- `FAIT_ENTREPRISE_AUTOMOBILE` : Répartition géographique des établissements

## 📁 Structure des Notebooks ETL

| Notebook | Objectif | Durée |
|----------|----------|-------|
| `1_Création_BDD_E5.ipynb` | Création du schéma complet (tables, PK, FK, index) | ~5 min |
| `2_data_DIM_TEMPS.ipynb` | Peuplement table temporelle (4 018 jours) | ~2 min |
| `3_data_from_carter_to_dwh.ipynb` | ETL base opérationnelle → DWH (dimensions pneus + faits prix) | ~15 min |
| `4_data_from_DL_CSV_to_dwh.ipynb` | ETL Data Lake → DWH (géographie + bornes recharge) | ~10 min |
| `5_data_from_DL_CSV_NAF_to_dwh.ipynb` | ETL fichier NAF Excel → DIM_ACTIVITE_NAF | ~3 min |
| `6_data_from_CSV_DIM_ENTREPRISE_to_dwh.ipynb` | ETL fichier SIRENE CSV → DIM_ENTREPRISE | ~8 min |
| `7_data_FAIT_ENTREPRISE_AUTOMOBILE.ipynb` | Création table de faits entreprises avec agrégations | ~5 min |
| `8_data_qualite__.ipynb` | Tests de qualité (59 tests automatisés) | ~3 min |

## 🚀 Installation et Configuration

### **Prérequis**
- Python 3.8+
- Drivers ODBC 17 for SQL Server
- Accès à Azure SQL Database et Azure Data Lake Gen2

### **Installation des dépendances**
```bash
pip install pandas pyodbc python-dotenv azure-storage-blob tqdm openpyxl xlrd
```

### **Configuration du fichier `.env`**
```env
DB_SERVER_DWH=votre-serveur.database.windows.net
DB_DATABASE_DWH=DWH_E5_projet_AUTO
DB_USERNAME_DWH=votre-username
DB_PASSWORD_DWH=votre-password
STORAGE_ACCOUNT_NAME=votre-storage
STORAGE_ACCOUNT_KEY=votre-key
```

## ▶️ Exécution des Notebooks

**Ordre d'exécution obligatoire :**
1. Notebook 1 (création schéma)
2. Notebook 2 (dimension temps)
3. Notebooks 3 à 7 (chargement données) - **ordre flexible**
4. Notebook 8 (validation qualité)

## 📊 Volumétrie Finale

- **Dimensions** : 103 967 lignes
- **Faits** : 24 263 lignes
- **Taux de qualité** : 98.3% (58/59 tests OK)

## 🎓 Compétences Validées

- **C14** : Conception modèle dimensionnel (schéma en étoile)
- **C15** : Garantie qualité des données (tests automatisés, intégrité référentielle)
- **C16** : Optimisation performances (44 index, stratégie SCD Type 2)