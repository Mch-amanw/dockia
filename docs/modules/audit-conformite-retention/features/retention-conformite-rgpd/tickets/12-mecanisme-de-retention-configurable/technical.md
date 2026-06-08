## Ticket : Mécanisme de rétention configurable – Spécification Technique

### 1. Modèle de données

#### 1.1 Table `retention_policies`
Utilisation de la table existante :
- `tenant_id` (UUID, unique, indexé)
- `document_retention_days` (integer, > 0)
- `result_retention_days` (integer, > 0)
- `audit_retention_days` (integer, > 0)
- `updated_at` (timestamp)

Contraintes :
- Validation applicative des valeurs positives.
- Toute modification déclenche un enregistrement dans `audit_logs`.

---

#### 1.2 Extension logique table `documents`
Champs utilisés :
- `status` (enum : active, archived, pending_deletion, deleted)
- `created_at` (timestamp)
- `deletion_scheduled_at` (timestamp nullable)
- `deleted_at` (timestamp nullable)

Index recommandés :
- `(tenant_id, status)`
- `(tenant_id, created_at)`
- `(tenant_id, deletion_scheduled_at)`

---

### 2. API de configuration

#### 2.1 Endpoints
- `GET /tenants/{id}/retention-policy`
- `PUT /tenants/{id}/retention-policy`

Contraintes :
- Authentification OAuth2/OIDC + JWT.
- Vérification RBAC (admin client ou plateforme).
- Vérification stricte du `tenant_id`.
- Journalisation via Audit Logger (event_type: retention_policy_updated).

---

### 3. Retention Manager

#### 3.1 Nature
Service planifié (cron interne ou worker dédié), exécuté périodiquement.

#### 3.2 Algorithme principal
1. Charger les `retention_policies`.
2. Pour chaque tenant :
   - Calculer la date limite = `now() - document_retention_days`.
   - Sélectionner les documents :
     - `status = active` ou `archived`
     - `created_at < date_limite`
   - Mettre à jour `status = pending_deletion`.
3. Pour chaque document `pending_deletion` :
   - Supprimer le fichier du stockage S3.
   - Si succès :
     - Mettre `status = deleted`.
     - Renseigner `deleted_at`.
     - Journaliser (event_type: document_deleted_auto).
   - Si échec :
     - Log technique.
     - Ne pas passer en `deleted`.

Traitement par batch (taille configurable).

---

### 4. Suppression des résultats structurés

Selon `result_retention_days` :
- Identifier les résultats liés à des documents expirés.
- Suppression logique ou physique en base.
- Journalisation (event_type: result_deleted_auto).

La suppression doit respecter les contraintes transactionnelles avec le document.

---

### 5. Gestion des logs d’audit

Selon `audit_retention_days` :
- Identifier les entrées `audit_logs` avec `created_at < seuil`.
- Suppression par batch.
- Respect de l’isolation par `tenant_id`.

Aucune suppression ne doit concerner des logs en dessous du seuil configuré.

---

### 6. Cohérence transactionnelle

- Utiliser des transactions DB pour :
  - Passage en `pending_deletion`.
  - Mise à jour finale en `deleted`.
- La suppression S3 doit être confirmée avant validation finale.
- Idempotence : relancer le worker ne doit pas provoquer d’erreur si le fichier est déjà supprimé.

---

### 7. Performance & Scalabilité

- Traitement paginé (LIMIT/OFFSET ou curseur).
- Index sur colonnes de filtrage.
- Paramètre de taille de batch configurable.
- Exécution non bloquante pour le pipeline principal.

---

### 8. Sécurité

- Filtrage systématique par `tenant_id`.
- Aucun accès cross-tenant.
- Aucun secret S3 dans les logs.
- Toutes les actions critiques journalisées.

Le mécanisme doit rester découplé des services IA et interagir uniquement via identifiants de documents et résultats.