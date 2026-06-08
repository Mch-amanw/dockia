## Spécification Technique – Pipeline asynchrone de traitement

### 1. Architecture technique

Composants impliqués :
- Broker de messages (queue).
- Workers asynchrones.
- Base PostgreSQL (`jobs`, `job_events`).
- Stockage S3.
- Services internes : OCR, Extraction, Scoring.
- Service Webhook.

Le pipeline est implémenté comme un consommateur de messages publié lors de la création d’un job.

---

### 2. Flux technique détaillé

#### 2.1 Consommation
1. Worker consomme un message contenant `job_id`.
2. Vérification idempotente :
   - Si job déjà `PROCESSING` ou terminal → abandon sécurisé.

#### 2.2 Passage en traitement
- Mise à jour du statut → `PROCESSING`.
- Enregistrement événement dans `job_events`.
- Enregistrement `started_at`.

#### 2.3 Exécution des étapes

1. Téléchargement du document depuis S3.
2. Appel module OCR.
3. Appel module Extraction.
4. Appel module Détection & Scoring.
5. Sauvegarde des résultats structurés et scores.
6. Enregistrement version des modèles et règles utilisées.

Chaque étape doit :
- Gérer ses exceptions.
- Retourner un objet structuré.
- Logger les métriques de durée.

---

### 3. Gestion des erreurs

- Exceptions interceptées au niveau worker.
- Mise à jour :
  - `status = FAILED`
  - `error_message` renseigné.
- Publication éventuelle en dead-letter queue après échec des retries.

Retries :
- Configurables (ex : 3 tentatives).
- Backoff exponentiel recommandé.

---

### 4. Idempotence

- Vérification systématique du statut avant exécution.
- Les écritures de résultats doivent être atomiques.
- Utilisation de transactions DB pour :
  - Mise à jour statut.
  - Sauvegarde résultats.
  - Historisation événement.

Objectif : éviter double traitement en cas de redelivery du message.

---

### 5. Webhook

Déclenchement après transaction de finalisation (`COMPLETED` ou `FAILED`).

Contenu minimal payload JSON :
```json
{
  "jobId": "uuid",
  "documentId": "uuid",
  "tenantId": "uuid",
  "status": "COMPLETED",
  "scoreGlobal": 78
}
```

- Appel HTTP POST.
- Timeout configurable.
- Retries avec journalisation.

---

### 6. Observabilité

Métriques à exposer :
- Temps total de traitement.
- Temps par étape.
- Taux d’échec.
- Nombre de retries.

Logs structurés incluant :
- `job_id`
- `tenant_id`
- étape en cours
- durée

---

### 7. Sécurité

- Validation du `tenant_id` avant accès aux ressources.
- Accès S3 limité par préfixe tenant.
- Communications internes sécurisées (TLS si externe).

---

### 8. Scalabilité

- Workers stateless.
- Scaling horizontal basé sur la profondeur de la queue.
- Compatible orchestration Docker/Kubernetes.

Objectif : supporter plusieurs milliers de documents/jour/client.

---