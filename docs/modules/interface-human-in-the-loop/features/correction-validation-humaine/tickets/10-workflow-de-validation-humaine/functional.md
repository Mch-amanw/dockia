## 1. Objectif du ticket

Implémenter le **workflow complet de validation humaine** dans le module Human-in-the-Loop (HITL), incluant :

- La correction des champs extraits ;
- La prise de décision (validation / rejet / escalade) ;
- La journalisation systématique des actions humaines ;
- La mise à jour cohérente du statut du document dans le pipeline.

Ce workflow formalise la transition entre le traitement automatique (OCR, extraction, scoring) et la décision humaine finale, conformément aux règles de gestion définies au niveau du module et du projet.

---

## 2. Déclenchement du workflow

Le workflow est applicable uniquement aux documents :

- En statut `pending_review` ;
- Appartenant au `tenant_id` de l’utilisateur connecté ;
- Pour lesquels l’utilisateur dispose d’un rôle autorisé (Analyste, Superviseur, Administrateur client).

Lorsqu’un analyste accède à un document en `pending_review`, celui-ci peut passer au statut `in_review` si le mécanisme d’assignation est actif.

---

## 3. Étapes du workflow

### 3.1 Consultation des résultats automatiques

L’utilisateur visualise :

- Le document original ;
- Les données extraites ;
- Les scores globaux et sous-scores ;
- Les signaux de fraude et facteurs explicatifs.

Aucune modification automatique des données n’est réalisée à cette étape.

---

### 3.2 Correction des champs

L’analyste peut :

- Modifier un champ extrait existant ;
- Ajouter une valeur manquante si le champ est prévu dans la structure du type de document.

Règles :

- Les données d’extraction originales restent immuables.
- Chaque correction crée un enregistrement distinct historisé.
- Plusieurs corrections peuvent être réalisées avant décision.
- Les corrections ne déclenchent pas de recalcul automatique du score (MVP).

Les corrections sont immédiatement visibles dans l’interface comme « valeur courante », tout en conservant l’accès à la valeur initiale.

---

### 3.3 Prise de décision

Une décision est obligatoire pour clôturer la revue.

Décisions possibles selon rôle :

- `validated`
- `rejected`
- `escalated` (si rôle autorisé)

Règles de gestion :

- La décision est associée à un utilisateur identifié.
- Un commentaire peut être requis selon la configuration du tenant.
- Une décision crée un enregistrement immuable.
- Toute modification ultérieure nécessite une nouvelle décision (si autorisée par les règles métier).
- Une décision humaine prévaut toujours sur le scoring automatique.

---

### 3.4 Mise à jour du statut

Transitions autorisées :

- `pending_review` → `in_review` (prise en charge)
- `pending_review` ou `in_review` → `validated`
- `pending_review` ou `in_review` → `rejected`
- `pending_review` ou `in_review` → `escalated`

La transition est atomique et cohérente avec l’enregistrement de la décision.

---

### 3.5 Journalisation et audit

Chaque action suivante génère un enregistrement traçable :

- Correction de champ
- Ajout de commentaire
- Décision (validation, rejet, escalade)
- Changement de statut

Les éléments historisés incluent :

- Identité utilisateur
- Horodatage
- Tenant
- Référence document
- Contexte technique (versions modèle et règles associées au document)

Aucune action ne peut être supprimée ou modifiée a posteriori.

---

## 4. Contraintes fonctionnelles

- Isolation stricte multi-tenant.
- Respect des rôles RBAC.
- Immutabilité des données historiques.
- Priorité de la décision humaine sur l’automatique.
- Conformité aux exigences d’audit et traçabilité définies au niveau projet.

---

## 5. Dépendances

- Module Authentification & RBAC
- Module Pipeline (gestion des statuts)
- Module Extraction structurée
- Module Scoring fraude
- Module Audit & traçabilité
- Base de données PostgreSQL

Ce ticket ne modifie pas le moteur IA ni les algorithmes de scoring.