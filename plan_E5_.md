# 🎯 PLAN COMPLET DU PROJET E5 - DATA WAREHOUSE
## Aligné sur les compétences de certification C13, C14, C15

---

## 📋 **PHASE 1 : ANALYSE MÉTIER ET INVENTAIRE DES DONNÉES**
**🎓 Compétence visée : C13 (préparation) + Exigence certification "produire une liste des données nécessaires"**

### **Étape 1.1 : Définition des besoins métier**
- Identifier les cas d'usage : Site web automobile + Tableaux de bord décisionnels
- Définir les questions métier auxquelles le DWH doit répondre
- Lister les KPI attendus (prix pneus, densité bornes, garages auto par zone)

### **Étape 1.2 : Inventaire exhaustif des sources de données**
**📄 Livrable : "Liste des données nécessaires aux analyses envisagées"**

| Source | Localisation | Usage DWH |
|--------|--------------|-----------|
| Produit, Caracteristiques, Dimensions | Azure SQL | Dimensions + Faits Prix |
| DimensionsParModel | Azure SQL | Dimension Véhicule |
| USER_API | Azure SQL | Agrégat comptage |
| SIRENE (42M lignes) | Data Lake Parquet | Dimension Entreprise (filtrée auto) |
| NAF | Data Lake XLS | Dimension Activité |
| Bornes IRVE | Data Lake CSV | Fait Bornes + Dimension Borne |
| Communes France | Data Lake CSV | Dimension Géographie |
| ParuVendu brut | Data Lake CSV | Enrichissement Véhicule |

### **Étape 1.3 : Identification des axes d'analyse (Dimensions)**
- **DIM_TEMPS** : Jour, Mois, Trimestre, Année, Saison
- **DIM_PRODUIT_PNEU** : Descriptif, Marque, Note
- **DIM_CARACTERISTIQUES_PNEU** : Saisonnalité, Type véhicule, Consommation, Bruit
- **DIM_DIMENSIONS_PNEU** : Largeur, Hauteur, Diamètre, Charge, Vitesse
- **DIM_VEHICULE** : Marque, Modèle, Année, Finition, Energie
- **DIM_GEOGRAPHIE** : Hiérarchie Région → Département → Commune (avec INSEE, population, coordonnées)
- **DIM_ENTREPRISE** : SIRET, SIREN, Dénomination, NAF, Adresse (filtré secteur automobile)
- **DIM_ACTIVITE_NAF** : Code NAF, Libellés, Hiérarchie Section/Division
- **DIM_BORNE_RECHARGE** : Type, Puissance, Opérateur, Statut

### **Étape 1.4 : Identification des indicateurs (Faits)**
- **FAIT_PRIX_PNEU** : Prix, Date scraping (historisation quotidienne)
- **FAIT_DISPONIBILITE_BORNE** : Nombre bornes, Puissance totale (snapshot mensuel)
- **FAIT_ENTREPRISE_AUTOMOBILE** : Nombre établissements actifs par zone (snapshot mensuel)

### **Étape 1.5 : Définition de l'architecture des datamarts**
- **DATAMART_PNEUMATIQUES** : Analyses prix/caractéristiques pneus
- **DATAMART_MOBILITE_ELECTRIQUE** : Analyses bornes de recharge
- **DATAMART_ENTREPRISES_AUTOMOBILE** : Analyses garages/entreprises auto
- **Tables transverses** : DIM_TEMPS, DIM_GEOGRAPHIE (partagées)

---

## 📐 **PHASE 2 : MODÉLISATION DIMENSIONNELLE**
**🎓 Compétence visée : C13 - Modéliser la structure des données**
**📄 Livrable : "Modélisations logiques et physiques de l'entrepôt de données et des datamarts"**

### **Étape 2.1 : Création du Modèle Logique de Données (MLD)**
- Schéma en étoile (star schema) pour chaque datamart
- Identification des clés de substitution (surrogate keys) pour toutes les dimensions
- Diagramme des relations Faits ↔ Dimensions
- Documentation des cardinalités et dépendances fonctionnelles

### **Étape 2.2 : Stratégie de gestion de l'historisation (SCD)**
- **SCD Type 0** : DIM_TEMPS, DIM_ACTIVITE_NAF (données de référence figées)
- **SCD Type 1** : DIM_BORNE_RECHARGE (écrasement du statut)
- **SCD Type 2** : DIM_PRODUIT_PNEU (historisation des prix avec Date_Debut/Date_Fin/Flag_Actuel), DIM_ENTREPRISE (historisation état administratif)

### **Étape 2.3 : Création du Modèle Physique de Données (MPD)**
- Définition des types de données SQL Server précis (INT, VARCHAR, DECIMAL, DATE)
- Scripts DDL de création de toutes les tables (Dimensions + Faits)
- Contraintes d'intégrité (PRIMARY KEY, FOREIGN KEY, NOT NULL, CHECK)
- Stratégie d'indexation :
  - Index clustered sur clés de substitution des dimensions
  - Index columnstore sur tables de faits (performances requêtes analytiques)
  - Index non-clustered sur colonnes de filtrage fréquentes

### **Étape 2.4 : Conception des agrégats pré-calculés (optionnel)**
- Tables d'agrégats pour accélérer les requêtes BI fréquentes
- Exemple : AGG_PRIX_MOYEN_PAR_DIMENSION_MOIS

### **Étape 2.5 : Validation du modèle**
- Revue du modèle avec les exigences métier (Phase 1)
- Vérification de la couverture de tous les KPI
- Documentation des choix de modélisation

---

## 🏗️ **PHASE 3 : CRÉATION DE L'INFRASTRUCTURE DWH**
**🎓 Compétence visée : C14 - Créer un entrepôt de données**
**📄 Livrables : "Configuration des outils" + "Configuration des accès"**

### **Étape 3.1 : Choix de la solution technique Azure**
- Comparaison Azure Synapse Analytics vs Azure SQL Database
- Justification du choix (coût, performances, contraintes projet)
- Dimensionnement des ressources (DTU, storage)

### **Étape 3.2 : Déploiement de l'infrastructure (IaC)**
- Création du script Terraform pour :
  - Resource Group dédié DWH
  - Azure SQL Server / Synapse Workspace
  - Bases de données DWH + Datamarts
  - Réseau virtuel et règles firewall
- Automatisation via GitHub Actions

### **Étape 3.3 : Création des bases de données et schémas**
- Base principale : `DWH_AUTOMOBILE`
- Schémas organisés par datamart :
  - `dbo.DIM_*` (dimensions transverses)
  - `pneumatique.*` (datamart pneus)
  - `mobilite.*` (datamart bornes)
  - `entreprise.*` (datamart sociétés auto)

### **Étape 3.4 : Exécution des scripts DDL**
- Création de toutes les tables de dimensions (9 tables)
- Création de toutes les tables de faits (3 tables)
- Création des index et contraintes
- Validation de la structure

### **Étape 3.5 : Configuration de la sécurité et des accès**
**📄 Livrable : "Configuration des accès aux données"**

#### **Accès aux données sources (exigence certification)**
- Configurer les Linked Services Azure Data Factory vers :
  - Azure SQL Database (tables opérationnelles Produit, Caracteristiques, etc.)
  - Azure Data Lake Gen2 (conteneurs bronze-data, data-gouv, gold-data)
  - Azure Purview (catalogue métadonnées)
- Gestion des identités managées (Managed Identity)
- Stockage sécurisé des credentials (Azure Key Vault)

#### **Accès pour les équipes d'analyse (exigence certification)**
- Création des groupes Azure AD :
  - `DWH_Analystes_BI` : Lecture tous datamarts
  - `DWH_Equipe_Web` : Lecture datamarts Pneumatique + Mobilité
  - `DWH_Data_Engineers` : Full access
  - `DWH_Admins` : Propriétaire base de données
- Attribution des rôles SQL :
  - `db_datareader` pour les analystes
  - `db_datawriter` + `db_datareader` pour les Data Engineers
- Configuration Row-Level Security (RLS) si nécessaire par région

### **Étape 3.6 : Configuration de la supervision**
- Azure Monitor : Métriques performances (DTU, stockage, latence)
- Alertes sur échecs de chargement ETL
- Logs d'accès et d'audit

---

# 🎯 PLAN COMPLET DU PROJET E5 - DATA WAREHOUSE (SUITE)

---

## 🔄 **PHASE 4 : DÉVELOPPEMENT DES PROCESSUS ETL** (SUITE)
**🎓 Compétence visée : C15 - Intégrer les ETL nécessaires**

### **Étape 4.4 : Développement ETL Faits** (SUITE)

#### **ETL 4.4.2 : Chargement FAIT_DISPONIBILITE_BORNE**
- Source : `bornes_irve_hdf_YYYYMMDD.csv` (Data Lake)
- Transformations :
  - Jointure avec DIM_BORNE_RECHARGE
  - Jointure avec DIM_GEOGRAPHIE (via coordonnées GPS ou code commune)
  - Jointure avec DIM_TEMPS
  - Calcul métriques : Nombre_Bornes, Puissance_Totale_kW
  - Agrégation par zone géographique
- Mode chargement : Snapshot mensuel (TRUNCATE/INSERT)
- Sortie : `DWH_AUTOMOBILE.mobilite.FAIT_DISPONIBILITE_BORNE`

#### **ETL 4.4.3 : Chargement FAIT_ENTREPRISE_AUTOMOBILE**
- Source : DIM_ENTREPRISE (déjà chargée)
- Transformations :
  - Jointure avec DIM_ACTIVITE_NAF
  - Jointure avec DIM_GEOGRAPHIE
  - Jointure avec DIM_TEMPS
  - Filtrage établissements actifs uniquement
  - Calcul métriques : Nombre_Etablissements, Estimation_Salaries (depuis tranche_effectifs)
- Mode chargement : Snapshot mensuel
- Sortie : `DWH_AUTOMOBILE.entreprise.FAIT_ENTREPRISE_AUTOMOBILE`

### **Étape 4.5 : Programmation des traitements de nettoyage**
**📄 Livrable : "Programmation des traitements appliqués aux données"**

- **Nettoyage SIRENE** : Suppression valeurs NULL critiques, normalisation adresses, validation SIRET
- **Nettoyage NAF** : Parsing format XLS, suppression lignes vides, validation codes
- **Nettoyage Bornes** : Validation coordonnées GPS, suppression doublons, standardisation noms opérateurs
- **Nettoyage Communes** : Validation codes INSEE, cohérence population/superficie
- **Nettoyage Pneus** : Validation prix (>0), dimensions cohérentes, notes format correct

### **Étape 4.6 : Configuration des zones de sortie ETL**
**📄 Livrable : "Configuration des zones de sorties des ETL"**

| Zone | Localisation | Type données | Rôle |
|------|--------------|--------------|------|
| **Bronze** | Data Lake `bronze-data` | Données brutes | Stockage initial sans transformation |
| **Silver** | Data Lake `gold-data/silver` | Données nettoyées | Après nettoyage et validation |
| **Gold** | Data Lake `gold-data/gold` | Données enrichies | Après transformations métier |
| **DWH Tables** | Azure SQL `DWH_AUTOMOBILE` | Tables dimensionnelles | Zone finale consommation |

### **Étape 4.7 : Orchestration et planification**
- Création pipelines Azure Data Factory :
  - **Pipeline_Dimensions** : Chargement quotidien dimensions (priorité 1)
  - **Pipeline_Faits_Pneus** : Chargement quotidien fait prix (priorité 2)
  - **Pipeline_Faits_Bornes** : Chargement mensuel (jour 1 du mois)
  - **Pipeline_Faits_Entreprises** : Chargement mensuel (jour 5 du mois)
- Configuration triggers temporels (schedules)
- Gestion des dépendances entre pipelines
- Mécanisme de retry en cas d'échec

### **Étape 4.8 : Gestion des erreurs et logging**
- Tables de contrôle ETL :
  - `ETL_LOG_EXECUTION` : Historique exécutions (date, durée, statut, lignes traitées)
  - `ETL_LOG_ERREURS` : Détail des erreurs (timestamp, pipeline, message, ligne source)
- Notifications email en cas d'échec
- Dashboard monitoring temps réel (Azure Monitor)

---

## ✅ **PHASE 5 : TESTS ET VALIDATION**
**🎓 Compétence visée : C15 (qualité des données) + Exigence certification "organiser la phase de test"**
**📄 Livrable : "Plan de test" + "Rapport de tests"**

### **Étape 5.1 : Définition du plan de test**
- Tests unitaires par ETL
- Tests d'intégration end-to-end
- Tests de performance
- Tests de qualité des données
- Tests de sécurité/accès

### **Étape 5.2 : Tests unitaires des transformations**
- Validation chaque transformation ETL individuellement
- Jeux de données de test contrôlés
- Vérification des règles métier (ex: prix > 0, codes NAF valides)

### **Étape 5.3 : Tests d'intégrité référentielle**
- Vérification clés étrangères (Faits → Dimensions)
- Absence de valeurs orphelines
- Contraintes NOT NULL respectées
- Unicité des clés de substitution

### **Étape 5.4 : Tests de volumétrie**
- Comptage lignes source vs destination
- Vérification cohérence agrégats
- Détection de pertes de données
- Exemple : `COUNT(*) FROM Produit` = `COUNT(*) FROM FAIT_PRIX_PNEU WHERE Date = TODAY`

### **Étape 5.5 : Tests de qualité des données**
- Détection valeurs NULL anormales
- Détection doublons
- Validation formats (dates, codes postaux, emails)
- Cohérence des données (ex: population > 0, superficie > 0)

### **Étape 5.6 : Tests de performance**
- Temps de chargement ETL (objectif < 30 min pour chargement quotidien)
- Temps de réponse requêtes analytiques (objectif < 5 sec pour requêtes BI)
- Optimisation index si nécessaire

### **Étape 5.7 : Tests de sécurité**
- Vérification accès par groupe AD
- Test blocage accès non autorisés
- Validation RLS (Row-Level Security) si implémenté

### **Étape 5.8 : Tests de bout en bout (E2E)**
- Simulation chargement complet : Source → Bronze → Silver → Gold → DWH
- Validation données accessibles depuis Power BI / Site Web
- Test requêtes métier réelles

### **Étape 5.9 : Documentation des résultats de test**
- Tableau de synthèse : Tests passés/échoués
- Actions correctives pour tests échoués
- Validation par responsable métier

---

## 📚 **PHASE 6 : DOCUMENTATION TECHNIQUE**
**🎓 Compétence visée : C13, C14, C15 (toutes) + Exigence certification "rédiger la documentation technique"**
**📄 Livrable : "Documentation technique complète du DWH"**

### **Étape 6.1 : Rédaction du dictionnaire de données**
- Description détaillée de toutes les tables (Dimensions + Faits)
- Pour chaque colonne : Nom, Type, Longueur, Nullable, Description métier, Exemple
- Glossaire des termes métier

### **Étape 6.2 : Documentation des modèles de données**
- Schémas visuels (MLD/MPD) au format image/PDF
- Légende des conventions (clés, relations, cardinalités)
- Justification des choix de modélisation

### **Étape 6.3 : Documentation des flux ETL**
- Diagramme de flux pour chaque pipeline
- Description étape par étape des transformations
- Règles de gestion appliquées
- Fréquence d'exécution et fenêtres de traitement

### **Étape 6.4 : Guide d'utilisation pour les analystes**
- Comment se connecter au DWH (Power BI, Excel, Python)
- Requêtes SQL types pour chaque cas d'usage métier
- Exemples de jointures entre Faits et Dimensions
- Bonnes pratiques de requêtage

### **Étape 6.5 : Documentation d'exploitation**
- Procédures de supervision (Azure Monitor, logs)
- Procédure de redémarrage en cas d'échec ETL
- Procédure de restauration (backups)
- Contacts support et escalade

### **Étape 6.6 : Documentation de sécurité**
- Matrice des accès (rôles, groupes AD, permissions)
- Procédure d'ajout d'un nouvel utilisateur
- Politique de gestion des credentials (Key Vault)

### **Étape 6.7 : Schéma d'architecture technique global**
- Diagramme complet : Sources → Data Lake → ETL → DWH → Consommateurs
- Technologies utilisées à chaque niveau
- Flux de données et dépendances

---

## 🔍 **PHASE 7 : RETOUR D'EXPÉRIENCE**
**🎓 Compétence visée : C14 + Exigence certification "formaliser un retour d'expérience des outils techniques"**
**📄 Livrable : "Retour d'expérience technique"**

### **Étape 7.1 : Analyse des choix technologiques**

#### **Azure SQL Database vs Azure Synapse Analytics**
- **Choix retenu** : [À compléter selon ton choix]
- **Cohérence avec contraintes projet** : Budget, volumétrie, complexité requêtes
- **Avantages** : Facilité déploiement, coût, intégration Azure, performances
- **Difficultés rencontrées** : Limitations DTU, temps chargement gros volumes, courbe apprentissage

#### **Azure Data Factory**
- **Cohérence** : Orchestration native Azure, intégration Data Lake/SQL
- **Avantages** : Interface visuelle, triggers, monitoring intégré, pas de serveur à gérer
- **Difficultés** : Debuggage pipelines complexes, gestion erreurs, coût calcul

#### **Azure Databricks (pour SIRENE)**
- **Cohérence** : Traitement distribué Spark nécessaire pour 42M lignes
- **Avantages** : Performance sur gros volumes, langage Python/SQL familier
- **Difficultés** : Coût cluster, configuration réseau, optimisation partitions Parquet

#### **Terraform (IaC)**
- **Cohérence** : Reproductibilité, versioning, automatisation CI/CD
- **Avantages** : Déploiement rapide, documentation as code, rollback facile
- **Difficultés** : Syntaxe HCL, gestion state file, permissions Azure

### **Étape 7.2 : Analyse des performances obtenues**
- Temps de chargement ETL vs objectifs
- Temps de réponse requêtes vs SLA
- Optimisations appliquées (index, partitionnement)
- Volumétrie traitée vs capacité infrastructure

### **Étape 7.3 : Identification des points d'amélioration**
- Automatisation tests qualité données
- Mise en place CDC (Change Data Capture) pour éviter full load
- Implémentation data lineage complet (Azure Purview)
- Migration vers architecture Lambda (temps réel + batch)

### **Étape 7.4 : Recommandations pour évolutions futures**
- Ajout nouvelles sources (API temps réel constructeurs auto)
- Création nouveaux datamarts (Assurance auto, Financement)
- Implémentation Machine Learning (prédiction prix pneus)
- Migration Progressive vers Synapse si volumes explosent

### **Étape 7.5 : Bilan compétences acquises**
- Modélisation dimensionnelle (schéma étoile, SCD Type 2)
- Maîtrise écosystème Azure Data (ADF, Databricks, SQL, Data Lake)
- Optimisation performances requêtes analytiques
- Gestion projet Data Engineering end-to-end

---

## 📦 **SYNTHÈSE DES LIVRABLES POUR LA CERTIFICATION**

| **Exigence certification** | **Livrable** | **Phase** | **Compétence** |
|----------------------------|--------------|-----------|----------------|
| Produire liste des données nécessaires | Document inventaire sources + besoins métier | Phase 1 | C13 |
| Modélisations logiques et physiques | MLD + MPD + Scripts DDL | Phase 2 | C13 |
| Configurer outils DWH | Scripts Terraform + Config Azure | Phase 3 | C14 |
| Configurer accès équipes analyse | Matrice accès AD + Rôles SQL | Phase 3 | C14 |
| Configurer accès sources | Linked Services ADF + Key Vault | Phase 3 | C14 |
| Organiser phase de test | Plan de test + Rapport résultats | Phase 5 | C15 |
| Rédiger documentation technique | Documentation complète DWH | Phase 6 | C13/C14/C15 |
| Formaliser retour d'expérience | REX outils techniques | Phase 7 | C14 |
| Intégrer sources aux ETL | Pipelines ADF/Databricks | Phase 4 | C15 |
| Configurer zones sortie ETL | Architecture Bronze/Silver/Gold | Phase 4 | C15 |
| Programmer traitements données | Code Python/SQL transformations | Phase 4 | C15 |

---

