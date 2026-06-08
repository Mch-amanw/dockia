## Fonctionnalité : Pipeline asynchrone de traitement

### 1. Objectif métier
Mettre en œuvre un pipeline de traitement documentaire asynchrone permettant d’orchestrer automatiquement les étapes OCR, extraction structurée et détection/scoring de fraude après l’upload d’un document.

Cette fonctionnalité garantit :
- La scalabilité du traitement (découplage API / exécution).
- La robustesse face aux pics de charge.
- La traçabilité complète des étapes.
- La notification fiable des résultats aux clients (webhook ou API).

---

### 2. Périmètre
Inclus :
- Consommation des jobs créés lors de l’upload.
- Orchestration séquentielle des étapes :
  1. OCR
  2. Extraction des données
  3. Analyse anti-fraude
  4. Calcul du score global et sous-scores
- Mise à jour des statuts de job.
- Gestion des erreurs et retries.
- Déclenchement de webhook en fin de traitement.

Exclus :
- Implémentation interne des modèles OCR, extraction ou IA (gérés par leurs modules respectifs).
- Interface utilisateur Human-in-the-Loop.

---

### 3. Règles de gestion

#### 3.1 Déclenchement
- Le pipeline est déclenché automatiquement après création d’un job valide.
- Aucun traitement ne s’exécute de manière synchrone côté API.

#### 3.2 Séquencement
- Les étapes sont exécutées strictement dans l’ordre : OCR → Extraction → Scoring.
- Une étape ne démarre que si la précédente est terminée avec succès.

#### 3.3 Gestion des statuts
Transitions autorisées :
- `PENDING` → `QUEUED`
- `QUEUED` → `PROCESSING`
- `PROCESSING` → `COMPLETED`
- `PROCESSING` → `FAILED`

Chaque transition doit être historisée.

#### 3.4 Gestion des erreurs
- En cas d’échec d’une étape critique :
  - Le job passe à `FAILED`.
  - Un motif d’erreur est enregistré.
- Les retries sont configurables (nombre maximum et délai).
- Après dépassement du nombre maximal de tentatives, le job est marqué définitivement `FAILED`.

#### 3.5 Notification
- Si un webhook est configuré pour le tenant, il est déclenché lorsque le job atteint l’état `COMPLETED` ou `FAILED`.
- La notification inclut au minimum :
  - `job_id`
  - `document_id`
  - `tenant_id`
  - `status`
  - `score_global` (si disponible)
- En cas d’échec du webhook, un mécanisme de retry est appliqué.

#### 3.6 Isolation multi-tenant
- Les workers doivent respecter strictement l’isolation par `tenant_id`.
- Aucun job ne peut accéder aux données d’un autre tenant.

---

### 4. Critères d’acceptation
- ✅ Un document uploadé génère un job traité entièrement sans blocage API.
- ✅ Les statuts évoluent correctement et sont historisés.
- ✅ Les résultats sont persistés avant passage à `COMPLETED`.
- ✅ En cas d’erreur simulée (ex : échec OCR), le job passe à `FAILED` avec message.
- ✅ Le webhook est déclenché uniquement en fin de traitement.
- ✅ Le traitement moyen respecte l’objectif < 30 secondes pour la majorité des documents.

---

### 5. Dépendances fonctionnelles
- Module Upload & gestion des jobs.
- Module OCR.
- Module Extraction structurée.
- Module Détection & Scoring.
- Module Audit & Logging.
- Module Webhook.

---

## Contraintes
- Traitement 100% asynchrone.
- Traçabilité complète.
- Haute résilience et capacité de montée en charge.

---