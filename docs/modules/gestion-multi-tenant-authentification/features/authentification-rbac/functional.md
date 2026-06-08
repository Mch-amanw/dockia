## Authentification & RBAC

### 1. Objectif métier
Mettre en place un mécanisme d’authentification sécurisé et une gestion fine des autorisations (RBAC) afin de :
- Garantir que seuls les utilisateurs autorisés accèdent à la plateforme Dockia.
- Assurer l’isolation stricte des données entre tenants.
- Appliquer des règles d’accès cohérentes selon les rôles (administrateur, superviseur, analyste, auditeur).
- Répondre aux exigences de sécurité, d’auditabilité et de conformité (RGPD, traçabilité).

Cette fonctionnalité constitue le socle de sécurité transversal à l’ensemble des modules (documents, scoring, Human-in-the-Loop, configuration).

---

### 2. Périmètre

Inclus dans cette fonctionnalité :
- Authentification via OAuth2 / OpenID Connect.
- Émission et validation de tokens JWT pour l’accès API.
- Gestion des rôles (RBAC) définis dans le module parent.
- Application des règles d’autorisation sur les endpoints API.
- Journalisation des connexions et actions critiques liées à l’accès.

Non inclus :
- Gestion avancée des politiques d’accès dynamiques (ABAC).
- Gestion multi-tenant avancée par schéma ou base dédiée.

---

### 3. Rôles supportés (MVP)

- **Administrateur plateforme** : gestion globale des tenants.
- **Administrateur client** : gestion des utilisateurs et paramètres du tenant.
- **Superviseur** : supervision des décisions et validations.
- **Analyste** : revue, correction et validation/rejet de documents.
- **Lecteur / Auditeur** : accès en lecture seule.

Chaque utilisateur (hors administrateur plateforme) est rattaché à un unique `tenant_id`.

---

### 4. Règles de gestion

#### 4.1 Authentification
- Toute requête API (hors endpoints explicitement publics) doit être authentifiée.
- L’accès est conditionné à la validité du token JWT.
- Un utilisateur inactif ou suspendu ne peut pas s’authentifier.
- Les tentatives répétées échouées peuvent entraîner un blocage temporaire.

#### 4.2 Token JWT
- Le token contient au minimum : `user_id`, `tenant_id`, `role`, `exp`.
- Le `tenant_id` est dérivé du contexte d’authentification et ne peut pas être modifié par le client.
- Les tokens expirés doivent être rejetés.

#### 4.3 Autorisation (RBAC)
- Chaque endpoint API définit explicitement les rôles autorisés.
- L’accès à une ressource est conditionné à :
  - La validité du token.
  - La correspondance du `tenant_id`.
  - Le rôle autorisé pour l’action demandée.
- Toute tentative d’accès non autorisée est refusée et journalisée.

#### 4.4 Isolation multi-tenant
- Un utilisateur ne peut accéder qu’aux ressources de son `tenant_id`.
- Toute incohérence entre le `tenant_id` du token et celui de la ressource entraîne un refus d’accès.

---

### 5. Critères d’acceptation

- ✅ Un utilisateur authentifié reçoit un JWT valide permettant d’accéder aux endpoints autorisés.
- ✅ Un utilisateur sans rôle suffisant reçoit une erreur d’autorisation (403).
- ✅ Un token expiré ou invalide entraîne une erreur d’authentification (401).
- ✅ Un utilisateur d’un tenant A ne peut pas accéder aux ressources du tenant B.
- ✅ Les connexions réussies et échouées sont journalisées.
- ✅ Les actions sensibles (modification configuration, validation/rejet, gestion utilisateurs) sont tracées.

---

### 6. Impacts métiers

- Renforce la confiance des clients (banques, fintechs, assurances).
- Garantit la conformité réglementaire et la séparation stricte des données.
- Constitue une base indispensable pour le Human-in-the-Loop et l’audit complet des décisions.