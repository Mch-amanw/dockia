## 1. Portée technique

Implémentation backend du workflow de validation humaine via :

- Endpoints REST de correction, décision et commentaires ;
- Gestion transactionnelle des changements de statut ;
- Journalisation systématique des actions ;
- Contrôles stricts RBAC et multi-tenant.

Aucune modification du moteur IA ni recalcul automatique du score.

---

## 2. Gestion des statuts document

### 2.1 États concernés

États autorisés :

- `pending_review`
- `in_review`
- `validated`
- `rejected`
- `escalated`

### 2.2 Règles de transition

Implémenter une logique de transition contrôlée :

- Vérification du statut courant avant toute décision.
- Refus si transition invalide.
- Mise à jour effectuée dans la même transaction que la création de l’entrée `review`.

La logique doit être centralisée dans un service métier (ex: `ReviewService`) afin d’éviter la duplication dans les endpoints.

---

## 3. Implémentation des corrections

Endpoint :

`POST /reviews/documents/{id}/corrections`

Traitement :

1. Vérification JWT.
2. Vérification rôle autorisé.
3. Vérification appartenance au `tenant_id`.
4. Vérification que le document est en `pending_review` ou `in_review`.
5. Lecture de la valeur originale depuis `extraction_result.structured_data`.
6. Insertion en base dans `field_correction`.
7. Écriture d’un événement d’audit.

Contraintes :

- `field_name` doit exister dans la structure du document.
- `original_value` stockée telle qu’extraite.
- `corrected_value` stockée sans écraser la donnée initiale.

---

## 4. Implémentation de la décision

Endpoint :

`POST /reviews/documents/{id}/decision`

Traitement transactionnel :

1. Vérification JWT et rôle.
2. Vérification tenant.
3. Vérification statut courant.
4. Création entrée `review` (UUID, tenant_id, reviewer_id, decision, comment, created_at).
5. Mise à jour statut document.
6. Écriture événement audit.
7. Option : déclenchement webhook post-commit si configuré.

La transaction doit être atomique (rollback complet en cas d’erreur).

---

## 5. Journalisation & audit

Pour chaque action :

- Appel au module Audit.
- Enregistrement d’un événement structuré (type_action, document_id, user_id, tenant_id, timestamp, metadata JSON).

Les métadonnées doivent inclure :

- Version modèle extraction.
- Version modèle scoring.
- Règles appliquées (si disponibles dans `fraud_score`).

Aucune suppression ou modification d’événement n’est autorisée.

---

## 6. Sécurité

- Vérification systématique du `tenant_id` dans toutes les requêtes.
- Contrôle RBAC centralisé.
- Validation stricte des entrées via Pydantic.
- Protection contre injection via whitelist des champs modifiables.

---

## 7. Performance et robustesse

- Temps de réponse cible < 500 ms.
- Endpoints stateless.
- Transactions PostgreSQL pour cohérence.
- Index sur `document_id`, `tenant_id`, `created_at`.

---

## 8. Non-régression

- Aucun recalcul automatique du score.
- Aucune modification des tables `extraction_result` et `fraud_score`.
- Respect strict du modèle de données existant.

Ce ticket structure le workflow métier côté backend et garantit la cohérence fonctionnelle, la traçabilité complète et la conformité multi-tenant.