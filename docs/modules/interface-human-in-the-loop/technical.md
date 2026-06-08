## 1. Architecture du module

Le module HITL est implémenté comme une interface web connectée à l’API Backend (FastAPI).

Composants :
- Frontend web (SPA ou rendu serveur selon architecture retenue)
- API REST dédiée aux opérations de revue
- Intégration avec PostgreSQL
- Accès sécurisé au stockage S3 pour affichage des documents

Il repose sur les services existants sans dupliquer la logique métier du scoring.

---

## 2. Modèle de données (conceptuel)

### 2.1 Entités principales

- `document`
  - id
  - tenant_id
  - type
  - storage_path
  - created_at

- `extraction_result`
  - document_id
  - structured_data (JSON)
  - confidence_scores
  - model_version

- `fraud_score`
  - document_id
  - global_score
  - sub_scores (JSON)
  - applied_rules
  - model_version

- `review`
  - id
  - document_id
  - reviewer_id
  - decision (validated / rejected / escalated)
  - comment
  - created_at

- `field_correction`
  - id
  - document_id
  - field_name
  - original_value
  - corrected_value
  - corrected_by
  - corrected_at

Toutes les entités incluent `tenant_id` pour garantir l’isolation logique.

---

## 3. API REST dédiée

Endpoints typiques :

- `GET /reviews/documents` : liste paginée avec filtres
- `GET /reviews/documents/{id}` : détail d’un document
- `POST /reviews/documents/{id}/corrections` : soumission de corrections
- `POST /reviews/documents/{id}/decision` : validation ou rejet
- `POST /reviews/documents/{id}/comments` : ajout de commentaire

Contraintes :
- Authentification JWT (OAuth2/OIDC).
- Vérification systématique du `tenant_id`.
- Contrôle d’accès basé sur les rôles.

---

## 4. Sécurité

- Vérification des permissions à chaque requête.
- Accès aux documents via URL signées temporaires (S3 pre-signed URLs).
- Journalisation des actions utilisateur.
- Protection CSRF/XSS côté frontend.

Conformité RGPD :
- Respect des politiques de rétention.
- Suppression logique ou physique selon configuration tenant.

---

## 5. Intégration au pipeline

- Le statut du document passe à `pending_review` si seuil dépassé.
- Une fois décision prise : mise à jour du statut (`validated` ou `rejected`).
- Option de déclenchement d’un webhook après décision humaine.

Le module ne recalcul pas le score automatiquement, sauf si explicitement prévu dans l’évolution future.

---

## 6. Audit et traçabilité technique

- Chaque action crée un enregistrement immuable en base.
- Historique consultable via endpoint dédié.
- Référence explicite aux versions de modèles utilisées.

---

## 7. Contraintes non fonctionnelles

- Temps d’affichage d’une fiche document < 2 secondes (hors chargement fichier volumineux).
- Pagination obligatoire pour les listes.
- Compatible montée en charge multi-tenant.
- Interface responsive pour usage desktop prioritaire.

---

## 8. Dépendances techniques

- Backend FastAPI
- PostgreSQL
- Stockage S3 compatible
- Service d’authentification (OAuth2 / OIDC)
- Module Audit

Ce module est dépendant des résultats produits par le pipeline asynchrone mais reste découplé du moteur IA.