## Spécification technique – Audit & traçabilité complète

### 1. Composants impactés
- Backend FastAPI
- Audit Logger (service interne)
- PostgreSQL (table `audit_logs`)
- Model Registry (`model_versions`)
- Pipeline asynchrone (workers)
- Module Human-in-the-Loop
- API d’audit sécurisée

---

### 2. Modèle de données

#### 2.1 Table `audit_logs`
Utilisation conforme au module parent :
- id (UUID)
- tenant_id (UUID, indexé)
- job_id (UUID, nullable, indexé)
- document_id (UUID, nullable)
- user_id (UUID, nullable)
- event_type (string, indexé)
- event_payload (JSONB)
- model_version (string, nullable)
- rule_version (string, nullable)
- created_at (timestamp, indexé)

Contraintes :
- Aucune mise à jour métier après insertion.
- Index sur (tenant_id, created_at), (tenant_id, job_id).
- Payload structuré (JSONB) pour flexibilité et filtrage.

---

### 3. Audit Logger

#### 3.1 Intégration
- Service interne appelé par :
  - API d’upload
  - Worker de traitement
  - Moteur IA
  - Module HIL
- Enregistrement idéalement asynchrone pour ne pas bloquer le pipeline.

#### 3.2 Format standardisé
Chaque appel au logger doit fournir :
- event_type (enum interne recommandé)
- tenant_id
- context (job_id, document_id)
- metadata (payload JSON sérialisé)
- model_version / rule_version si applicable

---

### 4. Intégration au pipeline

Au démarrage du job :
- Récupération des versions actives des modèles.
- Persistance de ces versions dans le contexte du job.
- Injection automatique des versions dans les logs associés.

À chaque étape :
- Génération d’un événement dédié (ex: OCR_COMPLETED, SCORING_COMPLETED).

---

### 5. Sécurité et isolation
- Filtrage systématique par `tenant_id` dans les requêtes.
- Vérification RBAC avant accès aux endpoints d’audit.
- Journalisation des accès aux endpoints d’audit.
- Chiffrement TLS en transit.

---

### 6. Performance et scalabilité
- Écriture optimisée (bulk insert possible côté worker).
- Indexation adaptée pour recherche par document et période.
- Possibilité de partitionnement futur par date.
- L’audit ne doit pas augmenter significativement le temps de traitement (< 30s cible globale).

---

### 7. API d’accès
Endpoints REST sécurisés :
- GET /audit/documents/{id}
- GET /audit/jobs/{id}
- GET /audit/logs (avec filtres)
- GET /audit/export

Filtres supportés :
- date_from / date_to
- event_type
- user_id
- job_id
- document_id

---

### 8. Contraintes
- Conformité RGPD.
- Conservation conforme à la politique de rétention du tenant.
- Aucun couplage fort avec un modèle IA spécifique.

La fonctionnalité doit rester générique, extensible et compatible avec l’évolution future des moteurs IA et des règles métier.