## Ticket : Ajout du champ `tenant_id` sur toutes les entités métier

### 1. Objectif
Mettre en conformité le modèle de données de Dockia avec la stratégie d’isolation logique multi-tenant en ajoutant un champ obligatoire `tenant_id` à toutes les entités métier.

Ce ticket constitue une brique fondatrice de la séparation stricte des données entre clients.

---

### 2. Contexte
La plateforme Dockia est multi-tenant avec une isolation logique basée sur un identifiant `tenant_id`. Cette isolation doit être garantie au niveau du modèle de données afin d’éviter toute fuite inter-tenant.

Actuellement, certaines entités métier ne disposent pas encore du champ `tenant_id` ou ne l’imposent pas de manière systématique.

---

### 3. Périmètre fonctionnel

#### 3.1 Entités concernées
Le champ `tenant_id` doit être ajouté (ou rendu obligatoire si déjà présent) sur toutes les entités métier, notamment :

- Document
- Job
- ExtractionResult
- FraudScore
- Rule
- TenantConfiguration
- AuditLog
- Toute autre entité métier manipulant des données client

Cas particulier :
- `User` → `tenant_id` obligatoire sauf pour le rôle Administrateur plateforme.

---

#### 3.2 Règles de gestion

- Toute entité métier doit être associée à un tenant valide.
- Aucune entité métier ne peut exister sans `tenant_id` (sauf exception explicitement définie comme l’administrateur plateforme).
- Le `tenant_id` ne doit pas être librement défini par l’utilisateur via l’API : il est dérivé du contexte d’authentification.
- Toute création d’entité doit automatiquement associer le `tenant_id` courant.

---

### 4. Impact fonctionnel

- Garantit la conformité avec la fonctionnalité d’isolation logique des tenants.
- Prépare le filtrage systématique des requêtes par `tenant_id`.
- Réduit le risque de fuite de données inter-tenant.
- Permet l’activation future de mécanismes avancés (Row-Level Security, partitionnement).

Ce ticket est critique et bloquant pour toute mise en production multi-client.