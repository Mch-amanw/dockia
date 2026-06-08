## Ticket : Endpoint upload document

### 1. Objectif
Mettre en place un endpoint REST sécurisé permettant aux clients de Dockia de téléverser un document (PDF ou image) afin de déclencher son traitement via le pipeline asynchrone.

Cet endpoint constitue le point d’entrée principal du flux documentaire côté API et doit garantir :
- La validation des fichiers entrants.
- L’association obligatoire à un tenant.
- La création automatique d’un document et d’un job de traitement.
- Le stockage sécurisé du fichier dans le stockage objet (S3 compatible).
- Un retour immédiat sans traitement synchrone long.

---

### 2. Périmètre fonctionnel

Inclus :
- Upload de fichiers aux formats : PDF (natif ou scanné), JPG, JPEG, PNG.
- Authentification obligatoire via JWT.
- Association automatique au `tenant_id` issu du contexte d’authentification.
- Création d’un enregistrement `document`.
- Création d’un `job` avec statut initial `PENDING` (ou `QUEUED` selon implémentation).
- Publication du job dans la file asynchrone.
- Retour d’une réponse JSON contenant les identifiants.

Exclus :
- Traitement OCR.
- Extraction.
- Détection de fraude.
- Logique de scoring.
- Interface web.

---

### 3. Comportement attendu

#### 3.1 Requête
L’API doit accepter :
- Un fichier binaire (multipart/form-data).
- Éventuellement des métadonnées minimales (ex : type de document si requis par le pipeline).

Contraintes :
- Format strictement limité aux formats supportés.
- Taille maximale configurable.
- Requête obligatoirement authentifiée.

---

#### 3.2 Traitement fonctionnel

À la réception d’une requête valide :
1. Vérification de l’authentification et des permissions (RBAC).
2. Validation du format et de la taille du fichier.
3. Génération d’un identifiant unique pour le document.
4. Stockage du fichier dans le bucket S3 structuré par `tenant_id`.
5. Création d’un enregistrement en base pour :
   - Le document.
   - Le job associé.
6. Publication d’un message dans la queue pour traitement asynchrone.
7. Retour immédiat de la réponse JSON.

Le traitement OCR/IA ne doit jamais être exécuté dans le contexte de la requête HTTP.

---

### 4. Réponse API

La réponse doit inclure :
- `document_id`
- `job_id`
- `status` initial
- `created_at`

Le statut initial doit refléter l’état réel du job (`PENDING` ou `QUEUED`).

---

### 5. Règles de gestion

- Chaque document est obligatoirement associé à un unique `tenant_id`.
- Un document ne peut exister sans job associé.
- Aucun traitement bloquant long n’est autorisé côté API.
- Les documents doivent être chiffrés au repos.
- Les erreurs doivent être explicites sans exposer d’informations internes sensibles.
- Toute tentative d’upload non autorisée doit être rejetée.

---

### 6. Dépendances fonctionnelles

- Module Authentification & RBAC.
- Module Stockage (S3 compatible).
- Module Ingestion & Orchestration (publication en queue).
- Base PostgreSQL.
- Module Audit & Logging.

---

### 7. Cas d’erreur gérés

- Format non supporté.
- Fichier corrompu ou vide.
- Taille maximale dépassée.
- Token invalide ou expiré.
- Absence ou invalidité du `tenant_id`.
- Échec de stockage S3.
- Échec de création en base.

Dans ces cas, aucun job ne doit être publié en queue.