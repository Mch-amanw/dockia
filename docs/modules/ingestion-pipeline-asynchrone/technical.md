## Module : Ingestion & Pipeline Asynchrone – Spécification Technique

### 1. Architecture du module

Le module se compose de :

1. **API Ingestion (FastAPI)**
   - Endpoint upload.
   - Endpoint consultation statut.
   - Endpoint récupération résultats.

2. **Service de queue / broker**
   - File de messages pour jobs.
   - Découplage API / Workers.

3. **Workers asynchrones**
   - Consommation des jobs.
   - Orchestration des appels OCR, Extraction, Scoring.

4. **Base PostgreSQL**
   - Table `documents`
   - Table `jobs`
   - Table `job_events` (historique statuts)

5. **Stockage S3 compatible**
   - Bucket structuré par tenant.

---

### 2. Modèle de données (conceptuel)

#### documents
- id (UUID)
- tenant_id
- filename
- storage_path
- created_at
- retention_policy_reference

#### jobs
- id (UUID)
- document_id (FK)
- tenant_id
- status
- error_message (nullable)
- created_at
- started_at
- completed_at

#### job_events
- id
- job_id
- previous_status
- new_status
- timestamp
- metadata (JSON)

---

### 3. Flux technique détaillé

1. Upload API :
   - Authentification JWT.
   - Validation input.
   - Upload vers S3.
   - Création `document`.
   - Création `job` avec statut `PENDING`.
   - Publication message en queue.
   - Retour immédiat `job_id`.

2. Worker :
   - Récupération message.
   - Passage statut `PROCESSING`.
   - Téléchargement depuis S3.
   - Appel services OCR → Extraction → Scoring.
   - Sauvegarde résultats.
   - Passage statut `COMPLETED` ou `FAILED`.
   - Déclenchement webhook si configuré.

---

### 4. Contraintes techniques
- Traitement asynchrone obligatoire.
- Idempotence : un message traité ne doit pas produire plusieurs résultats.
- Gestion des retries configurable.
- Timeout configurable par étape.
- Logs structurés pour audit.

---

### 5. Sécurité
- TLS pour toutes communications.
- Chiffrement au repos côté stockage.
- Validation stricte du tenant_id.
- Vérification des permissions via RBAC.

---

### 6. Scalabilité
- Workers scalables horizontalement.
- Stateless API.
- Queue supportant montée en charge.
- Compatible orchestration Docker/Kubernetes.

---

### 7. Intégrations
- Module Auth (OAuth2 / OIDC).
- Module OCR.
- Module Extraction.
- Module Détection & Scoring.
- Module Audit.
- Système Webhook externe.

---

### 8. Observabilité
- Logs applicatifs.
- Métriques :
  - Nombre de jobs.
  - Temps moyen de traitement.
  - Taux d’échec.
- Traces des transitions d’état.

Objectif : garantir résilience, auditabilité et conformité aux exigences RGPD et multi-tenant.