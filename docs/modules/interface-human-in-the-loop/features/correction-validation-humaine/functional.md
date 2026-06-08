## 1. Objectif métier

La fonctionnalité **Correction & validation humaine** permet aux analystes et superviseurs d’intervenir sur les résultats automatiques générés par le pipeline (OCR, extraction, scoring), afin de :

- Corriger les données extraites si nécessaire ;
- Compléter des champs manquants ;
- Prendre une décision finale (validation ou rejet) ;
- Justifier cette décision via des commentaires.

Elle garantit que la décision finale reste sous contrôle humain lorsque requis par la configuration du tenant ou lorsque le score de risque dépasse un seuil défini.

---

## 2. Périmètre fonctionnel

La fonctionnalité couvre :

- La modification des champs extraits affichés dans la fiche document ;
- L’enregistrement des corrections avec traçabilité complète (avant/après) ;
- La prise de décision finale (validated / rejected / escalated selon rôle) ;
- L’ajout de commentaires liés au document et/ou à la décision ;
- La mise à jour du statut du document dans le pipeline.

Elle s’applique uniquement aux documents appartenant au `tenant_id` de l’utilisateur connecté.

---

## 3. Règles de gestion

### 3.1 Correction des champs

- Les champs modifiables correspondent aux données structurées extraites.
- Les valeurs originales issues de l’extraction ne sont jamais supprimées.
- Chaque modification génère un enregistrement distinct contenant :
  - Nom du champ
  - Valeur originale
  - Valeur corrigée
  - Identité de l’utilisateur
  - Horodatage
- Plusieurs corrections peuvent être effectuées sur un même document.
- Les corrections n’altèrent pas les données historiques d’extraction (principe d’immutabilité).

### 3.2 Validation / Rejet

- Seuls les rôles autorisés (Analyste, Superviseur, Administrateur client) peuvent prendre une décision.
- Une décision est obligatoire pour clôturer une revue.
- Les décisions possibles :
  - `validated`
  - `rejected`
  - `escalated` (si rôle autorisé)
- Aucun document n’est déclaré "fraude certaine" : la décision repose sur une analyse humaine d’un score de risque.
- Selon la configuration du tenant, un commentaire peut être obligatoire.
- Une fois la décision enregistrée, elle est historisée et non modifiable (nouvelle décision nécessaire pour tout changement, si autorisé par les règles métier).

### 3.3 Commentaires

- Les utilisateurs autorisés peuvent ajouter des commentaires horodatés.
- Les commentaires sont liés au document et visibles dans l’historique.
- Les commentaires ne peuvent pas être supprimés, uniquement complétés.

### 3.4 Mise à jour du statut

- Lorsqu’un document est en statut `pending_review`, il passe à :
  - `validated` après validation humaine ;
  - `rejected` après rejet ;
  - `in_review` lorsqu’un analyste commence la revue (si mécanisme d’assignation actif).
- Une décision humaine prévaut toujours sur la décision automatique.

---

## 4. Critères d’acceptation

1. Un analyste peut modifier un champ extrait et voir la modification apparaître immédiatement dans l’interface.
2. La valeur originale reste accessible dans l’historique.
3. Chaque correction est associée à un utilisateur et un horodatage.
4. Une décision (validation ou rejet) met à jour le statut du document.
5. Une décision est historisée avec le score et les versions de modèles associés.
6. Un utilisateur ne peut pas corriger ou valider un document d’un autre tenant.
7. Les commentaires sont visibles dans l’ordre chronologique et non modifiables a posteriori.

---

## 5. Dépendances fonctionnelles

- Résultats d’extraction disponibles.
- Score de fraude calculé.
- Authentification et gestion des rôles (RBAC).
- Module Audit & traçabilité pour historisation des actions.