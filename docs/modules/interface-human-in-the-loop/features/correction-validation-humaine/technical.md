## 1. Composants impactés

- Frontend HITL (vue détail document)
- API Backend FastAPI (endpoints de revue)
- Base de données PostgreSQL
- Module Audit & traçabilité
- Pipeline (mise à jour des statuts)

Aucune modification du moteur IA n’est requise dans le cadre du MVP.

---

## 2. Modèle de données

### 2.1 Table `field_correction`

Champs :
- id (UUID)
- tenant_id
- document_id (FK)
- field_name (string)
- original_value (JSONB ou TEXT)
- corrected_value (JSONB ou TEXT)
- corrected_by (user_id)
- corrected_at (timestamp)

Contraintes :
- Vérification du `tenant_id` cohérent avec le document.
- Index sur `document_id` et `tenant_id`.

### 2.2 Table `review`

Champs :
- id (UUID)
- tenant_id
- document_id (FK)
- reviewer_id
- decision (ENUM: validated, rejected, escalated)
- comment (TEXT)
- created_at (timestamp)

Contraintes :
- Une décision finale par cycle de revue.
- Historisation immuable (pas d’UPDATE destructif ; nouvelle entrée si nouvelle décision autorisée).

### 2.3 Table `comment` (si distincte)

- id (UUID)
- tenant_id
- document_id (FK)
- author_id
- content (TEXT)
- created_at (timestamp)

---

## 3. API REST

### 3.1 Corrections

`POST /reviews/documents/{id}/corrections`

Body (JSON) :
```
{
  "fieldName": "invoice_amount",
  "correctedValue": "1200.00"
}
```

Traitement :
- Vérification JWT et rôle.
- Vérification appartenance au tenant.
- Récupération de la valeur originale depuis `extraction_result`.
- Création d’un enregistrement `field_correction`.
- Retour 201 Created.

---

### 3.2 Décision

`POST /reviews/documents/{id}/decision`

Body :
```
{
  "decision": "validated",
  "comment": "Montant confirmé après vérification visuelle"
}
```

Traitement :
- Vérification rôle autorisé.
- Contrôle statut courant (`pending_review` requis).
- Création entrée `review`.
- Mise à jour du statut du document.
- Écriture d’un événement d’audit.
- Option : déclenchement webhook si configuré.

---

### 3.3 Commentaires

`POST /reviews/documents/{id}/comments`

- Vérification des droits.
- Insertion en base.
- Retour 201.

---

## 4. Sécurité

- Vérification systématique du `tenant_id` sur chaque requête.
- Contrôle d’accès RBAC au niveau endpoint.
- Journalisation des actions sensibles (correction, décision).
- Protection contre injections via validation stricte des champs modifiables.

---

## 5. Intégration pipeline

- Transition de statut gérée via transaction DB.
- Décision humaine prioritaire sur score automatique.
- Aucun recalcul automatique du score lors d’une correction (MVP).

---

## 6. Contraintes non fonctionnelles

- Temps de réponse API < 500 ms hors latence réseau.
- Opérations atomiques (transaction DB).
- Compatible scalabilité horizontale (stateless API).
- Données historisées conformes aux exigences RGPD et audit interne.

---

## 7. Dépendances techniques

- FastAPI (routing + validation Pydantic)
- PostgreSQL (transactions, JSONB)
- Service Auth (OAuth2/OIDC + JWT)
- Module Audit

La fonctionnalité reste découplée du moteur d’extraction et du moteur de scoring.