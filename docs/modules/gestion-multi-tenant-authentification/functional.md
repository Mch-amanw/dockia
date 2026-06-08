## Module : Gestion Multi-Tenant & Authentification

### 1. Rôle du module dans Dockia
Ce module garantit l’isolation logique des clients (tenants), la gestion des utilisateurs et des rôles, ainsi que les mécanismes d’authentification et d’autorisation sécurisés. Il constitue le socle de sécurité et de conformité de la plateforme Dockia.

Il permet :
- La séparation stricte des données entre clients.
- La gestion des accès basée sur des rôles (RBAC).
- L’authentification sécurisée via OAuth2 / OIDC et JWT.
- La compatibilité avec les environnements entreprise (SSO, SAML/OIDC).

---

### 2. Gestion Multi-Tenant

#### 2.1 Isolation des données
- Chaque entité métier (document, job, résultat, règle, configuration, utilisateur) est associée à un `tenant_id`.
- Les utilisateurs ne peuvent accéder qu’aux données de leur tenant.
- Aucun accès inter-tenant n’est autorisé.
- Les requêtes API doivent systématiquement être filtrées par `tenant_id`.

#### 2.2 Configuration spécifique par tenant
Chaque tenant peut configurer :
- Seuils de validation automatique (score).
- Règles de scoring personnalisées.
- Webhooks.
- Politique de rétention des documents.

Les paramètres sont isolés et non visibles par d’autres tenants.

---

### 3. Gestion des utilisateurs

#### 3.1 Rôles
Les rôles supportés dans le MVP :
- **Administrateur plateforme** : gestion globale des tenants.
- **Administrateur client** : gestion des utilisateurs et paramètres du tenant.
- **Superviseur** : supervision des décisions et validations.
- **Analyste** : revue et correction des documents.
- **Lecteur / Auditeur** : accès en lecture seule.

#### 3.2 Règles d’autorisation
- Les permissions sont déterminées par le rôle.
- Les actions sensibles (modification de règles, suppression, export) sont limitées aux rôles autorisés.
- Toute action utilisateur est journalisée (audit trail).

---

### 4. Authentification

#### 4.1 Méthodes supportées
- OAuth2 / OpenID Connect.
- JWT pour accès API.
- Compatibilité SAML/OIDC pour SSO entreprise.

#### 4.2 Sessions et tokens
- Authentification basée sur token JWT signé.
- Expiration configurable des tokens.
- Rafraîchissement via refresh token.

#### 4.3 Sécurité
- Aucun mot de passe stocké en clair.
- Gestion sécurisée des secrets.
- Blocage possible après tentatives répétées échouées.

---

### 5. Règles de gestion
- Toute requête API doit être authentifiée (hors endpoints publics définis).
- Le `tenant_id` est dérivé du contexte d’authentification.
- Aucun utilisateur ne peut appartenir à plusieurs tenants dans le MVP (sauf administrateur plateforme).
- Toute tentative d’accès non autorisée doit être tracée.

---

### 6. Audit et conformité
- Journalisation des connexions.
- Journalisation des actions critiques.
- Traçabilité complète pour conformité RGPD.

Ce module est critique pour garantir la sécurité, la conformité réglementaire et la confiance des clients de Dockia.