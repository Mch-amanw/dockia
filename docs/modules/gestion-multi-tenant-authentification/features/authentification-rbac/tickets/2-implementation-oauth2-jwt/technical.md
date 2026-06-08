## Implémentation technique – OAuth2 + JWT

### 1. Architecture cible
Implémentation dans le backend FastAPI avec :

- Service d’intégration OAuth2 / OIDC.
- Générateur de JWT interne.
- Middleware d’authentification.
- Dépendances FastAPI pour injection du contexte utilisateur.

L’API reste stateless (aucune session serveur obligatoire).

---

### 2. Intégration OAuth2 / OIDC

#### 2.1 Validation du token IdP
- Récupération du token depuis la requête d’authentification.
- Validation via :
  - Clé publique (JWKS) du fournisseur, ou
  - Endpoint d’introspection.
- Vérification minimale :
  - Signature
  - Expiration
  - Issuer
  - Audience (si configurée)

#### 2.2 Mapping utilisateur
- Extraction des claims nécessaires (email ou identifiant unique).
- Recherche de l’utilisateur dans la table `User`.
- Vérification :
  - `status = active`
  - Association à un `tenant_id` valide.
- Provisioning minimal possible si autorisé (création utilisateur à la première connexion).

---

### 3. Génération du JWT interne

#### 3.1 Algorithme
- Signature via clé secrète (HS256) ou clé asymétrique (RS256).
- Clé stockée via variable d’environnement sécurisée.

#### 3.2 Payload minimal
```json
{
  "sub": "<user_id>",
  "tenant_id": "<tenant_id>",
  "role": "<role>",
  "exp": "<timestamp>"
}
```

#### 3.3 Configuration
- Durée de validité configurable.
- Rotation de clé supportée (clé active configurable).

---

### 4. Middleware d’authentification

#### 4.1 Extraction
- Lecture de l’en-tête :
  `Authorization: Bearer <token>`

#### 4.2 Validation
- Vérification signature.
- Vérification expiration.
- Décodage sécurisé.

#### 4.3 Injection contexte
Ajout dans `request.state` ou dépendance FastAPI :
- `current_user`
- `tenant_id`
- `role`

En cas d’échec :
- 401 si token absent ou invalide.

---

### 5. Refresh Token (optionnel mais prévu)

Si activé :
- Génération d’un refresh token opaque.
- Stockage en base (`AuthSession` table).
- Endpoint dédié pour renouvellement.
- Révocation possible en base.

---

### 6. Journalisation & Audit

Création d’entrées `AuditLog` pour :
- Authentification réussie.
- Tentative échouée.
- Token invalide.

Champs utilisés :
- `user_id`
- `tenant_id`
- `action` (LOGIN_SUCCESS, LOGIN_FAILED, TOKEN_INVALID)
- `timestamp`
- `metadata` (IP, provider si pertinent)

---

### 7. Sécurité
- TLS obligatoire.
- Secrets via variables d’environnement.
- Protection contre attaques par rejeu via expiration courte.
- Validation stricte des algorithmes acceptés.
- Aucune donnée sensible exposée dans les réponses d’erreur.

---

### 8. Contraintes techniques
- Compatible PostgreSQL.
- Stateless pour scalabilité horizontale.
- Compatible future implémentation SSO entreprise.
- Ne doit pas introduire de dépendance forte bloquante.