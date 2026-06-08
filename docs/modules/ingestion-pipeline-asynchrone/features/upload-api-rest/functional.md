## Fonctionnalité : Upload & API REST

### 1. Objectif métier
Permettre aux clients (banques, fintechs, assurances, entreprises) d’intégrer Dockia dans leurs processus KYC/KYB via une API REST sécurisée pour :
- Soumettre des documents à analyser.
- Déclencher automatiquement un job de traitement.
- Suivre l’état d’avancement.
- Récupérer les résultats structurés et les scores de risque.

Cette fonctionnalité constitue le point d’entrée technique principal de la plateforme côté intégration client.

---

### 2. Périmètre
La fonctionnalité couvre :
- Upload de documents via API REST.
- Création automatique d’un job associé.
- Consultation du statut d’un job.
- Récupération des résultats une fois le traitement terminé.
- Respect strict de l’isolation multi-tenant.

Hors périmètre :
- Interface web (couverte par une autre fonctionnalité).
- Logique interne OCR / Extraction / Scoring.
- Configuration avancée des règles métier.

---

### 3. Endpoints fonctionnels

#### 3.1 Upload d’un document
Permet de soumettre un document pour traitement.

Entrées :
- Fichier (PDF, JPG, JPEG, PNG).
- Métadonnées minimales si nécessaires (ex : type de document si requis).

Comportement :
- Validation du format.
- Vérification des droits du tenant.
- Création d’un identifiant unique de document.
- Création d’un job associé.
- Retour immédiat du `job_id`.

Sortie :
- `job_id`
- `document_id`
- Statut initial (`PENDING` ou `QUEUED`).

Règles de gestion :
- Chaque requête doit être authentifiée.
- Le document est obligatoirement rattaché à un `tenant_id`.
- L’API ne bloque jamais en attente du traitement complet.

---

#### 3.2 Consultation du statut d’un job
Permet de connaître l’état d’avancement du traitement.

Sortie :
- `job_id`
- `status` (`PENDING`, `QUEUED`, `PROCESSING`, `COMPLETED`, `FAILED`)
- Timestamps associés (création, début, fin si applicable).
- Message d’erreur si `FAILED`.

Règles de gestion :
- Un tenant ne peut consulter que ses propres jobs.
- Le statut doit refléter l’état réel du pipeline.

---

#### 3.3 Récupération des résultats
Accessible uniquement lorsque le job est `COMPLETED`.

Sortie :
- Données structurées extraites.
- Score global (0–100).
- Sous-scores.
- Indicateurs explicatifs.
- Version des modèles utilisés.

Règles de gestion :
- Si le job n’est pas `COMPLETED`, retour d’une erreur métier cohérente.
- Les résultats sont strictement isolés par tenant.

---

### 4. Règles de gestion clés
- Aucun traitement synchrone long côté API.
- Toute requête doit être authentifiée (OAuth2/OIDC + JWT).
- Isolation stricte des données par `tenant_id`.
- Journalisation complète des appels (audit).
- Les erreurs doivent être explicites mais ne jamais exposer d’informations sensibles.

---

### 5. Critères d’acceptation
- ✅ Un document valide soumis via API génère un `job_id`.
- ✅ Le statut évolue correctement jusqu’à `COMPLETED` ou `FAILED`.
- ✅ Les résultats sont récupérables uniquement si le job est terminé.
- ✅ Un tenant ne peut accéder ni aux jobs ni aux documents d’un autre tenant.
- ✅ Les erreurs de format ou d’authentification sont correctement gérées.

---

### 6. Dépendances
- Module Authentification & RBAC.
- Module Stockage S3.
- Module Orchestration du pipeline.
- Module Audit & Logging.
- Module Détection & Scoring (pour les résultats).