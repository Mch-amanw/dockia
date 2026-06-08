## Spécification Technique – Isolation logique des tenants

### 1. Modèle de données

#### 1.1 Champ `tenant_id`
- Ajout obligatoire d’un champ `tenant_id` (UUID ou équivalent) sur :
  - Document
  - Job
  - ExtractionResult
  - FraudScore
  - Rule
  - TenantConfiguration
  - AuditLog
  - User (nullable uniquement pour admin plateforme)
- Index systématique sur `tenant_id` pour performance.

Contraintes :
- Clé étrangère vers table `Tenant`.
- Non nullable pour toutes les entités métier.

---

### 2. Middleware d’injection du tenant

#### 2.1 Extraction depuis JWT
Le middleware FastAPI doit :
- Valider le JWT.
- Extraire :
  - `user_id`
  - `tenant_id`
  - `role`
- Injecter ces informations dans le contexte de requête.

Le `tenant_id` issu du token prévaut toujours sur toute valeur fournie dans la requête.

---

### 3. Filtrage des accès aux données

#### 3.1 Niveau application
- Toutes les requêtes ORM doivent inclure un filtre automatique `tenant_id = current_tenant_id`.
- Mise en place recommandée :
  - Repository pattern ou couche d’abstraction garantissant le filtrage.
  - Interdiction d’accès direct aux tables sans filtre.

#### 3.2 Contrôle d’accès aux ressources
Lors d’un accès par identifiant (ex: GET /documents/{id}) :
- Vérifier que la ressource appartient au `tenant_id` courant.
- Sinon retourner 403 ou 404.

---

### 4. Isolation du stockage S3

#### 4.1 Organisation logique
Structure recommandée :
```
/{tenant_id}/documents/{document_id}
/{tenant_id}/exports/...
```

- Les clés d’objets incluent obligatoirement le `tenant_id`.
- Les URLs pré-signées doivent être générées uniquement après vérification du tenant.

---

### 5. Cas particulier : Administrateur plateforme
- Si `tenant_id` est null dans le token et rôle = ADMIN_PLATFORM :
  - Accès autorisé sous conditions explicites.
  - Obligation de journalisation renforcée.

---

### 6. Sécurité renforcée
- Tests unitaires couvrant :
  - Accès autorisé intra-tenant.
  - Accès refusé inter-tenant.
- Tests d’intégration simulant deux tenants distincts.
- Revue obligatoire des endpoints pour vérifier l’absence de fuite.

Évolution future possible :
- Activation de Row-Level Security (PostgreSQL).
- Migration vers schémas dédiés par tenant.

---

### 7. Dépendances
- Dépend du module Authentification (JWT valide requis).
- Dépend du modèle `Tenant`.
- Impacte tous les modules métiers (documents, scoring, audit, configuration).

Cette fonctionnalité est transversale et doit être implémentée avant toute montée en charge ou onboarding client.