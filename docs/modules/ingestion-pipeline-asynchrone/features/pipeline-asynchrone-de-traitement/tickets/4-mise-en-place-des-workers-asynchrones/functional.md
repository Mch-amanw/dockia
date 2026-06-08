## Ticket : Mise en place des workers asynchrones

### 1. Objectif
Mettre en place l’infrastructure et le mécanisme d’exécution des **workers asynchrones** responsables du traitement des jobs documentaires (OCR, extraction, détection & scoring) via un système de file de tâches.

Ce ticket couvre la mise en place du socle technique permettant :
- Le découplage complet entre l’API d’ingestion et le traitement.
- La consommation fiable des jobs.
- L’exécution séquentielle du pipeline.
- La gestion des erreurs et des retries.
- La montée en charge horizontale.

Il ne couvre pas l’implémentation interne des modules OCR, Extraction ou Scoring.

---

### 2. Périmètre fonctionnel

Inclus :
- Intégration d’un broker de messages (file de tâches).
- Publication automatique d’un message lors de la création d’un job.
- Implémentation d’un ou plusieurs workers consommateurs.
- Traitement séquentiel des étapes :
  1. Téléchargement document
  2. OCR
  3. Extraction
  4. Détection & scoring
- Mise à jour des statuts (`QUEUED`, `PROCESSING`, `COMPLETED`, `FAILED`).
- Historisation des transitions d’état.
- Gestion des erreurs et retries.
- Préparation au déclenchement du webhook en fin de traitement.

Exclus :
- Interface utilisateur.
- Configuration avancée des règles métier.
- Implémentation des modèles IA.

---

### 3. Comportement attendu

#### 3.1 Publication en file
- Lorsqu’un job est créé, un message contenant au minimum `job_id` et `tenant_id` est publié dans la file.
- Le statut passe de `PENDING` à `QUEUED`.

#### 3.2 Consommation
- Un worker consomme les messages.
- Il vérifie l’état du job avant traitement (idempotence).
- Il passe le job à `PROCESSING`.

#### 3.3 Exécution du pipeline
Le worker exécute les étapes dans l’ordre défini.
Chaque étape :
- Doit être isolée.
- Doit pouvoir remonter une erreur contrôlée.
- Doit journaliser sa durée.

#### 3.4 Finalisation
- Si toutes les étapes réussissent :
  - Les résultats sont persistés.
  - Le job passe à `COMPLETED`.
- En cas d’erreur :
  - Le job passe à `FAILED`.
  - Le message d’erreur est enregistré.

#### 3.5 Retries
- Les échecs techniques peuvent déclencher un retry configurable.
- Après dépassement du nombre maximal de tentatives, le job est définitivement marqué `FAILED`.

#### 3.6 Scalabilité
- Les workers doivent être stateless.
- Plusieurs instances doivent pouvoir consommer la file simultanément.

---

### 4. Règles de gestion
- Un job ne peut être traité qu’une seule fois.
- Aucun traitement ne doit bloquer l’API.
- Toutes les transitions doivent être historisées.
- L’isolation multi-tenant doit être respectée.
- Les résultats doivent être persistés avant passage à `COMPLETED`.

---

### 5. Dépendances
- Module Ingestion & création de jobs.
- Base de données PostgreSQL.
- Stockage S3.
- Modules OCR, Extraction, Détection & Scoring.
- Module Audit & Logging.
- Module Webhook (pour étape ultérieure de notification).