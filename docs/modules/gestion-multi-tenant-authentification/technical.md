## Spécification Technique – Gestion Multi-Tenant & Authentification

### 1. Architecture
Module intégré au backend FastAPI.

Composants :
- Middleware d’authentification JWT.
- Middleware d’injection du `tenant_id`.
- Service RBAC (gestion des permissions).
- Service d’intégration OAuth2 / OIDC.
- Couche d’accès aux données filtrée par `tenant_id`.

---

### 2. Modèle de données (logique)

#### 2.1 Entités principales
- `Tenant`
  - id
  - name
  - configuration (JSON)
  - retention_policy
  - created_at

- `User`
  - id
  - tenant_id (nullable pour admin plateforme)
  - email
  - role
  - status
  - created_at

- `AuthSession` (optionnel)
  - user_id
  - refresh_token
  - expiration

Toutes les entités métier doivent contenir un champ `tenant_id` indexé.

---

### 3. Authentification

#### 3.1 JWT
- Signature via clé secrète ou clé asymétrique.
- Contenu minimal du token :
  - user_id
  - tenant_id
  - role
  - expiration
- Vérification automatique via middleware FastAPI.

#### 3.2 OAuth2 / OIDC
- Intégration avec fournisseur d’identité externe.
- Validation du token via endpoint de vérification ou clé publique.
- Mapping des claims vers utilisateur interne.

#### 3.3 SSO Entreprise
- Compatibilité SAML/OIDC.
- Support du provisioning utilisateur minimal (création à la première connexion si autorisé).

---

### 4. Autorisation (RBAC)

- Implémentation via dépendances FastAPI.
- Vérification des rôles avant exécution des endpoints.
- Découplage logique entre authentification et autorisation.

Exemple :
- `@require_role("ADMIN_CLIENT")`
- `@require_role("ANALYST")`

---

### 5. Isolation multi-tenant

#### 5.1 Niveau application
- Filtrage systématique des requêtes par `tenant_id`.
- Interdiction d’accès si mismatch entre token et ressource.

#### 5.2 Niveau base de données
- Index sur `tenant_id`.
- Possibilité future d’implémenter Row-Level Security (PostgreSQL).

---

### 6. Sécurité
- Hashage des mots de passe (bcrypt ou équivalent).
- TLS obligatoire.
- Rotation des clés JWT.
- Protection contre attaques classiques (brute force, injection).

---

### 7. Audit
- Table `AuditLog` contenant :
  - user_id
  - tenant_id
  - action
  - resource_type
  - resource_id
  - timestamp
  - metadata

Logs requis pour :
- Connexions.
- Modifications de configuration.
- Actions de validation/rejet.

---

### 8. Contraintes et dépendances
- Dépend de PostgreSQL pour stockage.
- Dépend du module API REST.
- Interagit avec tous les modules métiers pour contrôle d’accès.

Ce module est critique et doit être conçu pour être évolutif vers une isolation plus forte (schéma ou base dédiée par tenant).