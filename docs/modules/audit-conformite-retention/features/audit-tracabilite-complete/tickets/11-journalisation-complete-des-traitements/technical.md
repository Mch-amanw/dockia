## Spécification technique – Journalisation complète des traitements

### 1. Intégration avec Audit Logger

Tous les composants suivants doivent appeler le service interne `AuditLogger` :
- API d’upload
- Worker de pipeline
- Moteur OCR
- Service d’extraction
- Moteur de scoring/fraude
- Module Human-in-the-Loop

Signature logique attendue :

```
logEvent(
  tenantId: UUID,
  eventType: string,
  context: {
    jobId?: UUID,
    documentId?: UUID,
    userId?: UUID,
    userRole?: string
  },
  payload: dict,
  modelVersion?: string,
  ruleVersion?: string
)
```

L’appel doit être non bloquant (asynchrone ou via file interne) afin de ne pas impacter le temps de traitement global.

---

### 2. Structure des événements

#### 2.1 Événements techniques
Exemples d’`event_type` :
- DOCUMENT_UPLOADED
- JOB_CREATED
- OCR_COMPLETED
- EXTRACTION_COMPLETED
- SCORING_COMPLETED
- WEBHOOK_SENT

Pour `SCORING_COMPLETED`, le `event_payload` doit contenir :

```
{
  "globalScore": number,
  "subScores": {
    "documentIntegrity": number,
    "businessConsistency": number,
    "identity": number,
    "visualAnomalies": number,
    "aiSuspicion": number
  },
  "triggeredRules": ["RULE_CODE_1", "RULE_CODE_2"]
}
```

Les versions des modèles doivent être injectées automatiquement depuis le contexte du job.

---

#### 2.2 Actions Human-in-the-Loop
Exemples d’`event_type` :
- FIELD_CORRECTED
- DOCUMENT_VALIDATED
- DOCUMENT_REJECTED
- COMMENT_ADDED
- RESULT_VIEWED
- RESULT_EXPORTED

Exemple de payload pour correction :

```
{
  "fieldName": "invoice_amount",
  "oldValue": "1000.00",
  "newValue": "1100.00"
}
```

---

### 3. Persistance en base

Utilisation de la table `audit_logs` existante :

Champs utilisés :
- id (UUID)
- tenant_id
- job_id
- document_id
- user_id
- event_type
- event_payload (JSONB)
- model_version
- rule_version
- created_at (timestamp UTC)

Contraintes :
- Aucune opération UPDATE ou DELETE métier autorisée.
- Index sur (tenant_id, created_at), (tenant_id, job_id), (event_type).

---

### 4. Injection des versions de modèles

Au démarrage du job :
- Récupération des versions actives depuis `model_versions`.
- Stockage dans le contexte du job (mémoire + persistance éventuelle).
- Réutilisation automatique lors des appels `logEvent`.

Les versions doivent correspondre exactement aux modèles utilisés pour le traitement.

---

### 5. Isolation multi-tenant

- Chaque événement doit inclure un `tenant_id` obligatoire.
- Les requêtes d’écriture doivent vérifier la cohérence tenant/job/document.
- Aucun log ne doit être inséré sans `tenant_id` valide.

---

### 6. Performance

- Les écritures doivent être optimisées (possibilité de batch côté worker).
- L’audit ne doit pas augmenter significativement le temps total du job.
- En cas d’échec d’écriture d’un log, un mécanisme de retry ou fallback doit être prévu sans bloquer le traitement principal.

---

### 7. Sécurité

- Données sensibles dans `event_payload` limitées au strict nécessaire.
- Chiffrement TLS en transit.
- Accès aux logs uniquement via endpoints sécurisés RBAC.

Ce ticket implémente le socle opérationnel de traçabilité complète des traitements et décisions dans Dockia.