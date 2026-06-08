## 1. Composants concernés

### 1.1 Backend (FastAPI)

Endpoint principal :

`GET /reviews/documents/{id}`

Doit retourner :

- Métadonnées du document (id, type, statut, created_at)
- structured_data (JSON)
- confidence_scores (JSON)
- global_score
- sub_scores (JSON)
- applied_rules
- anomalies détectées (si stockées séparément)
- model_version (extraction + scoring)
- tenant_id (utilisé uniquement pour contrôle interne)

Contraintes :

- Vérification JWT (OAuth2/OIDC).
- Vérification rôle via RBAC.
- Filtrage strict par `tenant_id`.
- Retour d’erreur normalisé si document inexistant ou non accessible.

Endpoint complémentaire :

`GET /documents/{id}/file-url`

- Génère une URL S3 pré-signée temporaire.
- Durée limitée.
- Vérification préalable du `tenant_id` et des permissions.

---

## 2. Accès aux données (PostgreSQL)

Tables impliquées :

- `document`
- `extraction_result`
- `fraud_score`

Logique :

- Jointure par `document_id`.
- Filtrage obligatoire par `tenant_id`.
- Sélection de la version active des résultats (pas de recalcul).
- Les champs JSON (`structured_data`, `confidence_scores`, `sub_scores`, `applied_rules`) sont retournés tels que persistés.

Aucune écriture en base dans ce ticket.

---

## 3. Frontend

### 3.1 Architecture des composants

- `DocumentViewer`
  - Consomme l’URL pré-signée.
  - Support PDF multi-pages et images.
  - Gestion zoom et navigation.

- `ExtractionPanel`
  - Rend dynamiquement les champs à partir de `structured_data`.
  - Associe chaque champ à son `confidence_score`.
  - Mise en évidence conditionnelle des faibles confiances.

- `ScorePanel`
  - Affiche `global_score`.
  - Visualisation des `sub_scores` (barres ou indicateurs).

- `ExplainabilityPanel`
  - Liste `applied_rules`.
  - Affiche anomalies et facteurs explicatifs.
  - Affiche `model_version`.

---

### 3.2 Gestion des états

Pour chaque appel API :

- loading
- error (403, 404, 500)
- success

Si le document est supprimé selon la politique de rétention :

- Le backend retourne un statut cohérent.
- Le frontend affiche un message explicite.

---

## 4. Sécurité

- Authentification via JWT.
- Vérification systématique du `tenant_id` côté backend.
- Contrôle RBAC avant accès aux données.
- Accès au fichier exclusivement via URL S3 pré-signée.
- Journalisation de l’accès à la fiche document (lecture).

Aucune donnée sensible ne doit être exposée en dehors du périmètre du tenant.

---

## 5. Contraintes non fonctionnelles

- Temps de réponse cible < 2 secondes pour les données (hors téléchargement du fichier).
- Chargement asynchrone du document et des panneaux pour optimiser la perception utilisateur.
- Compatible avec architecture multi-tenant à grande échelle.
- Aucun recalcul de score ou d’extraction n’est effectué dans cette vue.

---

## 6. Dépendances techniques

- Backend FastAPI
- PostgreSQL
- Stockage S3 compatible (URL pré-signée)
- Service OAuth2 / OIDC
- Module Audit & Traçabilité

La vue est strictement consommatrice des résultats persistés par le pipeline et reste découplée du moteur IA.