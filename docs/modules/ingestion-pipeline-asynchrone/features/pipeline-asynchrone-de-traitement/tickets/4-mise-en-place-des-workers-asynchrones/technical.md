## Spécification Technique – Mise en place des workers asynchrones

### 1. Choix d’architecture

Le système repose sur :
- Un broker de messages (compatible production).
- Des workers Python intégrés à l’écosystème FastAPI.
- PostgreSQL pour la persistance.
- S3 compatible pour stockage documentaire.

Le worker doit être déployable comme service conteneurisé indépendant.

---

### 2. Intégration Broker

#### 2.1 File de messages
- Une queue dédiée aux jobs de traitement documentaire.
- Message minimal :
```json
{
  "jobId": "uuid",
  "tenantId": "uuid"
}
```

#### 2.2 Publication
- À la création du job :
  - Transaction DB validée.
  - Publication du message.
  - Passage statut `PENDING` → `QUEUED`.

---

### 3. Implémentation du Worker

#### 3.1 Consommation
- Écoute continue de la queue.
- Récupération du message.
- Vérification DB :
  - Si statut terminal → ignore.
  - Sinon → passage à `PROCESSING`.

#### 3.2 Idempotence
- Vérification transactionnelle du statut.
- Blocage si job déjà traité.
- Protection contre redelivery du broker.

---

### 4. Orchestration interne

Pseudo-flux :

1. Télécharger document depuis S3 (via `storage_path`).
2. Appeler module OCR.
3. Appeler module Extraction.
4. Appeler module Détection & Scoring.
5. Persister :
   - Données extraites
   - Scores détaillés
   - Versions modèles/règles
6. Mettre à jour statut final.

Chaque étape doit être encapsulée dans une couche service.

---

### 5. Gestion des erreurs

- Try/except global autour du pipeline.
- En cas d’exception :
  - Mise à jour `status = FAILED`.
  - Enregistrement `error_message`.
  - Historisation dans `job_events`.

Retries :
- Nombre configurable.
- Backoff exponentiel recommandé.
- Possibilité dead-letter queue.

---

### 6. Transactions et atomicité

Les opérations suivantes doivent être atomiques :
- Sauvegarde des résultats.
- Mise à jour du statut.
- Création événement dans `job_events`.

Utilisation de transactions PostgreSQL.

---

### 7. Observabilité

Logs structurés obligatoires :
- `job_id`
- `tenant_id`
- étape
- durée
- résultat

Métriques exposées :
- Nombre de jobs traités.
- Durée moyenne.
- Taux d’échec.
- Nombre de retries.

---

### 8. Sécurité

- Validation stricte du `tenant_id`.
- Accès S3 restreint par préfixe tenant.
- Pas de données sensibles dans les logs.
- Communication sécurisée avec services externes.

---

### 9. Scalabilité & Déploiement

- Worker conteneurisé (Docker).
- Stateless.
- Scaling horizontal basé sur profondeur de queue.
- Compatible orchestration Kubernetes.

Objectif : permettre traitement de plusieurs milliers de documents/jour/client.

---

### 10. Intégration future Webhook

Prévoir un hook interne post-finalisation permettant d’appeler ultérieurement le module Webhook sans modifier l’architecture du worker.