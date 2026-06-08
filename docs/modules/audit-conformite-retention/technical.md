## Module : Audit, Conformité & Rétention – Spécification Technique

### 1. Architecture du module
Module transversal intégré au backend FastAPI.

Composants principaux :
- Audit Logger (service interne)
- Audit Repository (PostgreSQL)
- Model Registry (gestion versioning)
- Retention Manager (service planifié)
- Audit API (endpoints REST sécurisés)

---

### 2. Modèle de données (PostgreSQL)

#### 2.1 Table audit_logs
Champs principaux :
- id (UUID)
- tenant_id (UUID, indexé)
- job_id (UUID, nullable)
- document_id (UUID, nullable)
- user_id (UUID, nullable)
- event_type (string)
- event_payload (JSONB)
- model_version (string, nullable)
- rule_version (string, nullable)
- created_at (timestamp, indexé)

Contraintes :
- Append-only logique (pas d’update métier).
- Index sur tenant_id, created_at, job_id.

#### 2.2 Table model_versions
- id
- model_name
- version
- type (ocr, extraction, scoring, fraude)
- created_at
- metadata (JSONB)
- is_active (bool)

#### 2.3 Table retention_policies
- id
- tenant_id
- document_retention_days
- result_retention_days
- audit_retention_days
- updated_at

---

### 3. Audit Logger

Fonctionnement :
- Appelé par les services métier (pipeline, HIL, API).
- Enregistrement asynchrone recommandé pour ne pas bloquer le traitement.
- Sérialisation des données sensibles contrôlée.

Exigences :
- Tolérance aux pannes.
- Idempotence possible sur certains événements.

---

### 4. Versioning des modèles

- Lors du démarrage d’un job, récupération de la version active.
- Persistance de la version utilisée dans l’enregistrement du job.
- Association automatique aux entrées audit_logs.

Aucune suppression physique des versions historiques.

---

### 5. Retention Manager

Service planifié (cron ou worker dédié) :
- Identification des documents expirés.
- Suppression du fichier dans le stockage S3.
- Mise à jour du statut en base.
- Journalisation de la purge.

Respect :
- Isolation par tenant.
- Suppression par lot pour scalabilité.

---

### 6. API d’audit

Endpoints sécurisés (RBAC) :
- GET /audit/documents/{id}
- GET /audit/jobs/{id}
- GET /audit/logs?filters=
- GET /audit/export

Authentification via OAuth2/OIDC + JWT.

---

### 7. Sécurité

- Isolation stricte via tenant_id dans toutes les requêtes.
- Vérification systématique des permissions.
- Chiffrement TLS.
- Logs d’accès aux endpoints d’audit.

---

### 8. Scalabilité

- Indexation adaptée pour volumes élevés.
- Partitionnement possible des audit_logs par date.
- Purge par batch.
- Compatible avec plusieurs milliers de documents/jour/tenant.

---

### 9. Contraintes

- Ne pas dégrader le temps de traitement (< 30 secondes cible).
- Conformité RGPD.
- Conservation suffisante pour audit réglementaire.

Le module doit rester découplé des moteurs IA afin de permettre leur évolution indépendante.