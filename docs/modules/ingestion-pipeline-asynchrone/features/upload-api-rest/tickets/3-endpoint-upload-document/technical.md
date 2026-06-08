## Ticket : Endpoint upload document – Spécification technique

### 1. Endpoint REST

**Méthode** : `POST`  
**Route** : `/documents`  
**Content-Type** : `multipart/form-data`

Authentification :
- OAuth2 / OIDC.
- JWT signé.
- Extraction du `tenant_id` depuis le token.

RBAC :
- Vérification que le rôle utilisateur autorise l’upload.

---

### 2. Validation des entrées

Contrôles à implémenter :
- Présence du fichier.
- Extension et MIME type autorisés :
  - `application/pdf`
  - `image/jpeg`
  - `image/png`
- Taille maximale configurable (via variable d’environnement ou configuration).
- Vérification que le fichier n’est pas vide.

Les validations doivent être effectuées avant tout stockage permanent.

---

### 3. Stockage S3

- Génération d’un `document_id` (UUID).
- Construction d’un chemin structuré :
  `/{tenant_id}/{document_id}/{original_filename}`
- Upload vers bucket S3 compatible.
- Chiffrement au repos activé.
- Communication TLS obligatoire.

En cas d’échec d’upload :
- Aucun enregistrement en base ne doit être finalisé.

---

### 4. Persistance en base de données

#### Table `documents`
Insertion :
- id (UUID)
- tenant_id
- filename
- storage_path
- created_at
- retention_policy_reference (nullable)

#### Table `jobs`
Insertion :
- id (UUID)
- document_id (FK)
- tenant_id
- status = `PENDING`
- error_message = NULL
- created_at

#### Table `job_events`
Insertion :
- job_id
- previous_status = NULL
- new_status = `PENDING`
- timestamp
- metadata (JSON optionnel)

Les opérations doivent être transactionnelles côté base (transaction SQL).

---

### 5. Publication en queue

- Publication d’un message contenant au minimum :
  - `job_id`
  - `document_id`
  - `tenant_id`
- Publication uniquement après validation et insertion en base.
- Le message doit permettre un traitement idempotent côté worker.

Aucun appel aux modules OCR/Extraction/Scoring n’est autorisé dans ce endpoint.

---

### 6. Gestion des erreurs HTTP

Codes recommandés :
- 400 : validation échouée (format, taille, fichier manquant).
- 401 : non authentifié.
- 403 : non autorisé (RBAC).
- 500 : erreur interne (S3, DB, queue).

Les messages d’erreur :
- Ne doivent pas exposer de stacktrace.
- Doivent être logués côté serveur avec `tenant_id` si disponible.

---

### 7. Observabilité

Logs structurés incluant :
- `tenant_id`
- `document_id`
- `job_id`
- endpoint
- durée de traitement API

Métriques à exposer :
- Nombre d’uploads.
- Taux d’erreur.
- Latence moyenne.

---

### 8. Contraintes non fonctionnelles

- Endpoint stateless.
- Temps de réponse court (pas de dépendance IA synchrone).
- Compatible scalabilité horizontale (Docker/Kubernetes).
- Respect strict de l’isolation multi-tenant.

Ce ticket dépend :
- De la configuration S3.
- De la disponibilité PostgreSQL.
- Du broker de queue.
- Du module Auth.