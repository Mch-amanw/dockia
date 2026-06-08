## Implémentation OAuth2 + JWT

### 1. Objectif du ticket
Mettre en place un mécanisme d’authentification sécurisé basé sur OAuth2 / OpenID Connect et des tokens JWT permettant :

- L’authentification des utilisateurs de Dockia.
- L’émission de tokens JWT pour l’accès à l’API.
- La validation systématique des tokens sur les endpoints protégés.
- L’injection sécurisée du contexte utilisateur (`user_id`, `tenant_id`, `role`).

Ce ticket constitue le socle technique indispensable au RBAC et à l’isolation multi-tenant.

---

### 2. Périmètre fonctionnel

Inclus :
- Intégration avec un fournisseur d’identité OAuth2 / OIDC.
- Génération d’un JWT interne signé après authentification valide.
- Validation des JWT pour toutes les routes protégées.
- Gestion de l’expiration des tokens.
- Support optionnel du refresh token.
- Journalisation des événements d’authentification (succès / échec).

Non inclus :
- Gestion avancée des politiques d’accès (ABAC).
- Interface UI de gestion des utilisateurs.
- Intégration SAML complète (prévue ultérieurement).

---

### 3. Comportement attendu

#### 3.1 Authentification
- Un utilisateur s’authentifie via OAuth2 / OIDC.
- Le token fourni par l’IdP est validé.
- Si l’utilisateur est autorisé et actif, un JWT interne Dockia est généré.
- Le JWT est utilisé pour toutes les requêtes API suivantes.

#### 3.2 Structure du JWT
Le token doit contenir au minimum :
- `sub` : identifiant utilisateur
- `tenant_id`
- `role`
- `exp` : date d’expiration

Le `tenant_id` est déterminé côté serveur et ne peut pas être modifié par le client.

#### 3.3 Validation des requêtes API
- Toute requête vers un endpoint protégé doit inclure :
  `Authorization: Bearer <token>`
- Les tokens invalides, expirés ou mal signés sont rejetés.
- En cas de succès, le contexte utilisateur est disponible pour les contrôles RBAC et multi-tenant.

#### 3.4 Gestion de l’expiration
- Les tokens expirés sont refusés.
- Si activé, un mécanisme de refresh token permet d’obtenir un nouveau JWT.

#### 3.5 Journalisation
Les événements suivants doivent être journalisés :
- Connexions réussies.
- Tentatives échouées.
- Tokens invalides ou expirés.

Ces logs alimentent le module d’audit.

---

### 4. Contraintes fonctionnelles
- Toute route API est protégée par défaut sauf exception explicite.
- Le système doit être compatible multi-tenant.
- L’authentification ne doit pas exposer d’informations sensibles.
- Le mécanisme doit être compatible avec une architecture stateless.

---

### 5. Dépendances
- Module Gestion Multi-Tenant & Authentification.
- Tables `User`, `Tenant`, `AuditLog`.
- Middleware RBAC (utilisera le contexte injecté par ce ticket).