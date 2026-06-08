## Spécification Technique – Ajout du champ `tenant_id`

### 1. Modifications du schéma PostgreSQL

#### 1.1 Type et contrainte
- Type recommandé : `UUID`
- Référence : clé étrangère vers `tenant(id)`
- Contrainte : `NOT NULL` pour toutes les entités métier (sauf `user` si admin plateforme)
- Index obligatoire sur `tenant_id`

Exemple SQL générique :
```sql
ALTER TABLE document
ADD COLUMN tenant_id UUID NOT NULL REFERENCES tenant(id);

CREATE INDEX idx_document_tenant_id ON document(tenant_id);
```

Ces opérations doivent être réalisées via un système de migration (ex: Alembic).

---

### 2. Migration des données existantes

Si des données existent déjà :
- Attribution d’un `tenant_id` par défaut (tenant initial) défini explicitement.
- Script de migration garantissant l’absence de lignes avec `tenant_id` NULL.
- Vérification post-migration.

---

### 3. Mise à jour des modèles ORM (SQLAlchemy)

#### 3.1 Ajout du champ
Dans chaque modèle concerné :

```python
tenant_id = Column(UUID(as_uuid=True), ForeignKey("tenant.id"), nullable=False, index=True)
```

- Définir la relation ORM vers `Tenant` si nécessaire.
- Interdire toute création d’instance sans `tenant_id`.

---

### 4. Intégration avec le contexte d’authentification

- Le `tenant_id` doit être injecté via le middleware JWT.
- Les services métier ne doivent pas accepter un `tenant_id` fourni par le payload API.
- Le `tenant_id` utilisé lors de la persistance doit provenir du contexte sécurisé (`current_user.tenant_id`).

---

### 5. Cas particulier : table User

- `tenant_id` nullable uniquement si `role = ADMIN_PLATFORM`.
- Validation métier empêchant :
  - `tenant_id` NULL pour les autres rôles.

---

### 6. Performance et indexation

- Index B-Tree sur `tenant_id` pour chaque table.
- Vérifier les requêtes fréquentes (documents, jobs) pour s’assurer que les filtres futurs utiliseront l’index.

---

### 7. Sécurité

- Aucun accès direct à la base ne doit contourner la contrainte NOT NULL.
- Les contraintes de clé étrangère doivent empêcher la référence à un tenant inexistant.

---

### 8. Dépendances

- Dépend de l’existence de la table `tenant`.
- Pré-requis pour l’implémentation du filtrage systématique par `tenant_id`.
- Impacte tous les modules métiers (documents, scoring, audit, configuration, pipeline).

Ce ticket doit être implémenté avant toute logique avancée d’isolation ou mise en production multi-tenant.