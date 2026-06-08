## 1. Objectif métier

L’**Interface de revue analyste** permet aux utilisateurs habilités (analystes, superviseurs, administrateurs) d’analyser un document traité par le pipeline Dockia afin de :

- Visualiser le document original
- Examiner les données extraites automatiquement
- Comprendre le score global de risque et les sous-scores
- Identifier les signaux de fraude détectés
- Disposer des éléments explicatifs nécessaires à une prise de décision

Cette fonctionnalité constitue le cœur opérationnel du module Human-in-the-Loop (HITL) et vise à fournir une aide à la décision claire, explicable et auditée.

---

## 2. Périmètre

La fonctionnalité couvre uniquement l’affichage et la consultation détaillée d’un document donné. Elle inclut :

- Visualisation du document source (PDF/image)
- Affichage structuré des données extraites
- Affichage des scores (global + sous-scores)
- Présentation des signaux de fraude et facteurs explicatifs
- Accès à certaines métadonnées (type, date, statut, modèle utilisé)

Elle n’inclut pas :
- La modification des données (couverte par la fonctionnalité de correction)
- La prise de décision (couverte par la fonctionnalité de validation/rejet)

---

## 3. Description fonctionnelle détaillée

### 3.1 Accès à la fiche de revue

Un utilisateur accède à la fiche détaillée depuis la liste des documents à revoir.

Conditions :
- L’utilisateur doit être authentifié.
- Il doit appartenir au même `tenant_id` que le document.
- Il doit disposer d’un rôle autorisé (analyste, superviseur, administrateur, auditeur en lecture seule).

---

### 3.2 Visualisation du document

L’interface doit permettre :

- L’affichage du document original (PDF natif, PDF scanné ou image).
- La navigation multi-pages.
- Le zoom avant/arrière.
- Le défilement fluide.

Règles de gestion :
- Le document affiché est strictement celui stocké lors de l’upload.
- Aucun contenu ne peut être modifié depuis cette vue.
- L’accès est sécurisé et temporaire.

---

### 3.3 Affichage des données extraites

Les données sont présentées sous forme structurée, adaptée au type de document :

- Facture : numéro, date, montant, devise, fournisseur, client, IBAN, etc.
- Pièce d’identité : nom, prénom, date de naissance, numéro de document, date d’expiration, etc.
- Relevé bancaire : titulaire, IBAN, période, transactions clés, etc.

Pour chaque champ :
- Valeur extraite
- Niveau de confiance associé

Règles de gestion :
- Les champs à faible confiance peuvent être visuellement mis en évidence.
- Les données affichées correspondent à la version la plus récente enregistrée (incluant corrections éventuelles).
- Les valeurs originales restent accessibles via l’historique.

---

### 3.4 Affichage des scores et sous-scores

L’interface affiche :

- Score global de risque (0–100)
- Sous-scores par catégorie :
  - Intégrité documentaire
  - Cohérence métier
  - Identité
  - Anomalies visuelles
  - Suspicion de génération/modification par IA

Règles de gestion :
- Le score est présenté comme indicateur de risque, jamais comme certitude de fraude.
- Les seuils applicables sont ceux configurés pour le tenant.
- Les scores affichés correspondent à la version du modèle utilisée lors du traitement.

---

### 3.5 Signaux de fraude et explicabilité

L’interface doit présenter :

- La liste des règles déclenchées.
- Les anomalies détectées.
- Les facteurs ayant contribué au score.
- Le niveau de confiance global.
- La version des modèles et des règles appliquées.

Objectif :
- Permettre à l’analyste de comprendre pourquoi un document a obtenu ce score.

Règles de gestion :
- Les explications doivent être lisibles et interprétables.
- Les éléments affichés doivent être traçables et auditables.

---

## 4. Critères d’acceptation

- Un utilisateur autorisé peut accéder à la fiche d’un document de son tenant.
- Le document original s’affiche correctement avec navigation multi-pages.
- Les données extraites sont visibles avec leur niveau de confiance.
- Le score global et les sous-scores sont affichés.
- Les signaux de fraude et facteurs explicatifs sont visibles.
- Un utilisateur d’un autre tenant ne peut pas accéder au document.
- Les informations affichées correspondent exactement aux données enregistrées en base.

---

## 5. Dépendances fonctionnelles

- Liste des documents à revoir
- Module Extraction structurée
- Module Moteur de scoring
- Module Audit & traçabilité
- Module Authentification & RBAC
- Module Stockage documentaire

Cette fonctionnalité dépend de la disponibilité complète des résultats produits par le pipeline asynchrone.