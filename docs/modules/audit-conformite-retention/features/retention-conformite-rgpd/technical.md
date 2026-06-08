## Spécification Technique – Rétention & Conformité RGPD

### 1. Composants impactés

- Backend FastAPI (API configuration & suppression).
- Base PostgreSQL (tables retention_policies, documents, audit_logs).
- Stockage objet S3 compatible (suppression physique).
- Retention Manager (worker planifié).
- Audit Logger (journalisation des événements).

---

### 2. Modèle de données

#### 2.1 retention_policies
Utilisation de la table existante :
- tenant_id (clé unique)
- document_retention_days
- result_retention_days
- audit_retention_days
- updated_at

Contraintes :
- Index sur tenant_id.
- Historisation via audit_logs lors des modifications.

#### 2.2 documents (extension logique)
- status (enum : active, archived, pending_deletion, deleted)
- deletion_scheduled_at (timestamp nullable)
- deleted_at (timestamp nullable)

---

### 3. Retention Manager

Service planifié (cron interne ou worker dédié) :

Étapes :
1. Charger les politiques par tenant.
2. Identifier les documents dépassant document_retention_days.
3. Marquer en pending_deletion.
4. Supprimer physiquement le fichier du stockage S3.
5. Mettre à jour le statut en deleted.
6. Journaliser chaque étape via Audit Logger.

Contraintes :
- Traitement par batch pour éviter surcharge.
- Isolation stricte par tenant.
- Idempotence : relancer la purge ne doit pas produire d’incohérence.

---

### 4. Suppression à la demande

Endpoint sécurisé (RBAC) :
- DELETE /documents/{id}

Comportement :
- Vérification du tenant_id.
- Vérification des permissions (administrateur client ou plateforme).
- Suppression fichier S3.
- Mise à jour statut en base.
- Génération d’un audit_log (event_type = document_deleted_manual).

Option : anonymisation partielle des résultats si conservation requise.

---

### 5. Chiffrement

- Données en transit : TLS obligatoire.
- Documents au repos : chiffrement via capacités S3.
- Accès restreint aux buckets par politique IAM.

Aucune clé sensible ne doit être exposée dans les logs.

---

### 6. API de configuration

Endpoints :
- GET /tenants/{id}/retention-policy
- PUT /tenants/{id}/retention-policy

Contraintes :
- Authentification OAuth2/OIDC.
- Vérification RBAC.
- Journalisation de toute modification.

---

### 7. Sécurité & conformité

- Filtrage systématique par tenant_id dans toutes les requêtes.
- Journalisation des accès aux endpoints sensibles.
- Respect du principe de minimisation des données.
- Les logs d’audit ne peuvent être supprimés que selon audit_retention_days.

---

### 8. Performance & scalabilité

- Index sur created_at et status.
- Possibilité de partitionnement des tables volumineuses.
- Suppression par lots configurables.
- Conception compatible avec plusieurs milliers de documents/jour/tenant.

---

### 9. Contraintes

- Ne pas impacter le temps de traitement du pipeline (< 30 secondes cible).
- Garantir la cohérence transactionnelle entre suppression base et suppression S3.
- Assurer tolérance aux pannes (reprise sur erreur).

La fonctionnalité doit rester découplée du moteur IA et interagir uniquement via identifiants de documents et jobs.