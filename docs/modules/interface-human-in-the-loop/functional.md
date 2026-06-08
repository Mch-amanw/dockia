## 1. Rôle du module

Le module **Interface Human-in-the-Loop (HITL)** permet l’intervention humaine dans le processus de traitement documentaire de Dockia. Il offre une interface web dédiée aux analystes et superviseurs pour :

- Visualiser les documents traités
- Examiner les données extraites
- Comprendre les signaux de fraude et les scores générés
- Corriger les informations si nécessaire
- Valider ou rejeter un document
- Ajouter des commentaires et justifications

Ce module s’inscrit dans le pipeline asynchrone et intervient après l’extraction et le scoring, en particulier lorsque :
- Le score dépasse un seuil configurable nécessitant revue manuelle
- Une validation humaine est obligatoire selon la configuration du tenant

---

## 2. Fonctionnalités principales

### 2.1 Accès et périmètre
- Accès restreint aux utilisateurs authentifiés.
- Respect strict du multi-tenant : un utilisateur ne peut consulter que les documents de son `tenant_id`.
- Filtrage des droits selon les rôles (RBAC) :
  - Administrateur plateforme
  - Administrateur client
  - Superviseur
  - Analyste
  - Lecteur / Auditeur

---

### 2.2 Liste des documents à revoir

Interface listant les documents avec :
- Identifiant du document
- Type (facture, pièce d’identité, relevé bancaire)
- Date de soumission
- Statut du job
- Score global de risque
- Statut de revue (à traiter, en cours, validé, rejeté)
- Assignation éventuelle à un analyste

Fonctionnalités :
- Filtres (date, score, statut, type de document)
- Tri par score ou date
- Recherche par identifiant

---

### 2.3 Vue détaillée d’un document

La fiche document affiche :

#### a) Visualisation du document
- Affichage du PDF ou de l’image originale
- Zoom et navigation multi-pages

#### b) Données extraites
- Champs structurés selon le type de document
- Mise en évidence des champs à faible confiance
- Indication du niveau de confiance par champ

#### c) Scores et signaux de fraude
- Score global (0–100)
- Sous-scores par catégorie (intégrité, cohérence métier, identité, anomalies visuelles, suspicion IA)
- Liste des signaux ayant contribué au score
- Facteurs explicatifs associés

---

### 2.4 Correction des données

Les analystes peuvent :
- Modifier les champs extraits
- Ajouter des valeurs manquantes
- Corriger des erreurs OCR

Règles de gestion :
- Les valeurs originales sont conservées.
- Toute modification est historisée (avant/après).
- L’identité de l’utilisateur et l’horodatage sont enregistrés.
- Les corrections ne suppriment jamais les données initiales.

---

### 2.5 Validation et décision

Actions disponibles :
- Valider le document
- Rejeter le document
- Marquer comme nécessitant une escalade (si rôle superviseur)

Règles de gestion :
- Une décision doit être associée à un utilisateur identifié.
- Un commentaire peut être requis selon la configuration du tenant.
- La décision finale est historisée et liée à la version du modèle et aux règles appliquées.
- Aucun document n’est marqué comme « fraude certaine » : uniquement validation ou rejet basé sur un score de risque.

---

### 2.6 Commentaires et collaboration

- Ajout de commentaires horodatés.
- Visualisation de l’historique des échanges.
- Traçabilité complète des interactions.

---

### 2.7 Audit et historique

La fiche document doit inclure :
- Historique des traitements (OCR, extraction, scoring)
- Version des modèles utilisés
- Règles appliquées
- Historique complet des actions humaines

Ces informations sont accessibles en lecture selon le rôle (lecture seule pour auditeurs).

---

## 3. Règles de gestion spécifiques

- Isolation stricte des données par `tenant_id`.
- Seuils de revue automatique configurables par client.
- Les décisions humaines priment sur la décision automatique.
- Toute action humaine est traçable, non modifiable a posteriori.
- Les corrections peuvent être exploitées ultérieurement pour amélioration continue des modèles (sans modifier les données historiques).

---

## 4. Dépendances

- Module Authentification & RBAC
- Module Pipeline de traitement
- Module Extraction structurée
- Module Moteur de scoring
- Module Audit & traçabilité
- Module Stockage documentaire (S3)

Le module HITL dépend de la disponibilité des résultats générés par le pipeline.