## Spécification technique : Upload & API REST

### 1. Composants impactés

- API Backend (FastAPI).
- Base PostgreSQL (`documents`, `jobs`, `job_events`).
- Stockage objet S3 compatible.
- Service de queue (publication de messages).
- Module Auth (OAuth2 / OIDC + JWT).
- Module Webhook (déclenchement post-traitement).

---

### 2. Endpoints REST (conceptuels)

#### POST /documents
Fonction : Upload d’un document.

Traitement :
1. Validation JWT.
2. Extraction du `tenant_id` depuis le token ou contexte.
3. Validation format et taille.
4. Upload vers S3 (chemin structuré par tenant).
5. Création en base :
   - `documents`.
   - `jobs` (statut `PENDING`).
6. Publication d’un message en queue.
7. Retour JSON avec `job_id`.

Contraintes :
- Endpoint non bloquant.
- Timeout court côté API.
- Idempotence recommandée (clé idempotence optionnelle).

---

#### GET /jobs/{job_id}
Fonction : Consultation statut.

Traitement :
- Vérification appartenance tenant.
- Lecture en base (`jobs`).
- Retour statut + timestamps + erreur éventuelle.

---

#### GET /jobs/{job_id}/results
Fonction : Récupération résultats.

Traitement :
- Vérification appartenance tenant.
- Vérification statut `COMPLETED`.
- Lecture des résultats structurés (stockés en base ou stockage objet).
- Retour JSON structuré.

---

### 3. Modèle de données utilisé

Tables :
- `documents` : métadonnées document + référence S3.
- `jobs` : suivi du traitement.
- `job_events` : historique transitions.

Contraintes :
- Clé étrangère `document_id`.
- Index sur `tenant_id`.
- Index sur `status` pour monitoring.

---

### 4. Sécurité

- Authentification OAuth2/OIDC.
- JWT signé.
- Validation stricte du `tenant_id`.
- Vérification RBAC (rôle autorisé à uploader/consulter).
- Limitation taille fichier.
- Protection contre upload malveillant.
- TLS obligatoire.

---

### 5. Gestion des erreurs

Cas gérés :
- 400 : format invalide.
- 401/403 : non authentifié / non autorisé.
- 404 : job inexistant ou hors tenant.
- 409 : tentative accès résultats avant complétion.
- 500 : erreur interne.

Les erreurs doivent être loguées pour audit.

---

### 6. Performance et scalabilité

- API stateless.
- Publication asynchrone en queue.
- Aucun traitement IA exécuté dans l’API.
- Compatible scalabilité horizontale (Docker/Kubernetes).

Objectif : supporter plusieurs milliers de documents/jour/client.

---

### 7. Observabilité

- Logs structurés (tenant_id, job_id, endpoint).
- Métriques :
  - Nombre d’uploads.
  - Taux d’erreur.
  - Latence API.
- Corrélation possible avec logs worker.

---

### 8. Dépendances techniques

- Module Ingestion & Orchestration (publication queue).
- PostgreSQL.
- S3 compatible.
- Système de queue.
- Module Auth.