## Module : Ingestion & Pipeline Asynchrone

### 1. Rôle du module dans Dockia
Le module **Ingestion & Pipeline Asynchrone** est responsable de :
- La réception des documents (interface web et API REST).
- La création et gestion des jobs de traitement.
- L’orchestration du pipeline OCR → Extraction → Détection de fraude → Scoring.
- La gestion des statuts et notifications associées.

Il constitue le point d’entrée opérationnel du traitement documentaire et garantit un flux robuste, scalable et isolé par tenant.

---

### 2. Fonctionnalités principales

#### 2.1 Upload de documents
- Acceptation des formats : PDF (natif/scanné), JPG, JPEG, PNG.
- Upload via :
  - API REST sécurisée.
  - Interface web.
- Validation initiale :
  - Format supporté.
  - Taille maximale configurable.
  - Présence d’un `tenant_id` valide.
- Attribution d’un identifiant unique de document.
- Stockage du fichier dans le stockage objet (S3 compatible).

Règles de gestion :
- Chaque document est obligatoirement associé à un tenant.
- Aucun traitement n’est lancé sans création explicite d’un job.
- Les documents sont chiffrés au repos.

---

#### 2.2 Création et gestion des jobs
- Création automatique d’un **Job de traitement** lors de l’upload.
- Association Job ↔ Document ↔ Tenant.
- Attribution d’un identifiant unique de job.
- Statuts possibles :
  - `PENDING`
  - `QUEUED`
  - `PROCESSING`
  - `COMPLETED`
  - `FAILED`
- Historisation des transitions d’état.

Règles de gestion :
- Un job correspond à un document.
- Un job ne peut être traité qu’une seule fois.
- En cas d’erreur, le statut passe à `FAILED` avec motif enregistré.

---

#### 2.3 Orchestration du pipeline
Orchestration séquentielle des étapes suivantes :
1. OCR
2. Extraction structurée
3. Analyse anti-fraude
4. Calcul des scores
5. Stockage des résultats

Caractéristiques :
- Traitement asynchrone par défaut.
- Découplage entre API et workers.
- Mise à jour progressive du statut.

Règles de gestion :
- Les étapes sont exécutées dans l’ordre logique défini.
- Si une étape critique échoue, le job est marqué `FAILED`.
- Les versions des modèles et règles utilisées doivent être enregistrées.

---

#### 2.4 Notification et consultation
- Consultation du statut via API.
- Récupération des résultats lorsque le job est `COMPLETED`.
- Notification optionnelle via webhook configuré par tenant.

Règles de gestion :
- Les webhooks sont déclenchés uniquement lorsque le traitement est terminé.
- Les notifications doivent inclure : job_id, document_id, statut, score global.

---

#### 2.5 Isolation multi-tenant
- Isolation logique stricte via `tenant_id`.
- Aucun accès inter-tenant.
- Configuration spécifique par tenant (seuils, règles, webhooks, rétention).

---

### 3. Contraintes fonctionnelles
- Traitement cible < 30 secondes pour la majorité des documents.
- Support de pics de charge.
- Traçabilité complète des étapes.
- Aucune décision bloquante synchrone côté API.

---

### 4. Dépendances fonctionnelles
- Module Authentification & RBAC (contrôle d’accès).
- Module Stockage (S3).
- Module OCR.
- Module Extraction.
- Module Détection & Scoring.
- Module Audit & Logging.

---

### 5. Cas limites
- Document corrompu → statut `FAILED`.
- Timeout de traitement → statut `FAILED`.
- Erreur IA → journalisation + statut `FAILED`.
- Webhook indisponible → retry configurable.