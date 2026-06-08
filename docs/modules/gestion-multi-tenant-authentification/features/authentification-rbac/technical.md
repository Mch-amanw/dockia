## Spécification Technique – Authentification & RBAC

### 1. Composants impactés

- Backend FastAPI.
- Middleware d’authentification JWT.
- Service d’intégration OAuth2 / OIDC.
- Service RBAC (vérification des rôles).
- Base PostgreSQL (tables User, Tenant, AuditLog).

---

### 2. Authentification

#### 2.1 OAuth2 / OIDC
- Intégration avec un fournisseur d’identité (IdP).
- Validation des tokens via clé publique ou endpoint d’introspection.
- Mapping des claims externes vers :
  - `user_id`
  - `tenant_id`
  - `role`

Provisioning minimal possible à la première connexion si autorisé par configuration.

#### 2.2 JWT interne
- JWT signé (clé secrète ou asymétrique).
- Contenu minimal :
  - `sub` (user_id)
  - `tenant_id`
  - `role`
  - `exp`
- Vérification automatique via middleware FastAPI.
- Expiration configurable.
- Support des refresh tokens (stockés en base si implémentés).

---

### 3. Middleware & dépendances FastAPI

#### 3.1 Middleware d’authentification
- Extraction du token depuis l’en-tête `Authorization: Bearer <token>`.
- Validation signature et expiration.
- Injection dans le contexte de requête :
  - `current_user`
  - `tenant_id`
  - `role`

#### 3.2 Vérification RBAC
- Implémentation via dépendances FastAPI.
- Exemple logique :
  - `require_role(["ADMIN_CLIENT", "SUPERVISOR"])`
- Vérification avant exécution du handler.
- Retour HTTP :
  - 401 si non authentifié.
  - 403 si rôle insuffisant.

---

### 4. Modèle de données

#### Table `User`
- `id`
- `tenant_id` (nullable pour admin plateforme)
- `email`
- `role`
- `status` (active, suspended)
- `password_hash` (si authentification locale activée)
- `created_at`

#### Table `AuditLog`
- `user_id`
- `tenant_id`
- `action`
- `resource_type`
- `resource_id`
- `timestamp`
- `metadata` (JSON)

Index requis sur :
- `User.tenant_id`
- `AuditLog.tenant_id`

---

### 5. Isolation multi-tenant

- Le `tenant_id` est extrait uniquement du token validé.
- Toute requête aux repositories doit inclure un filtre explicite par `tenant_id`.
- Vérification supplémentaire si une ressource est chargée par ID :
  - Comparaison entre `resource.tenant_id` et `token.tenant_id`.

Évolution possible : activation future de Row-Level Security (PostgreSQL).

---

### 6. Sécurité

- Hashage des mots de passe via bcrypt (si login local).
- Rotation possible des clés JWT.
- TLS obligatoire.
- Limitation des tentatives de connexion (rate limiting côté API).
- Aucun secret exposé dans le code source.

---

### 7. Audit & traçabilité

Doivent être journalisés :
- Connexions réussies.
- Tentatives échouées.
- Création/modification/suppression d’utilisateur.
- Modification des rôles.
- Tentatives d’accès non autorisées.

Ces logs alimentent la table `AuditLog` et participent à la conformité RGPD et à l’audit global du système.

---

### 8. Contraintes & dépendances

- Dépend du module PostgreSQL.
- Interagit avec tous les autres modules via contrôle d’accès.
- Doit rester compatible avec architecture scalable (stateless API + JWT).
- Ne doit pas introduire de dépendance forte empêchant une évolution vers SSO entreprise complet.