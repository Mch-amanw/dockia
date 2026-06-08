## 1. Objectif du ticket

Développer l’interface de visualisation d’un document dans le cadre de la revue analyste (HITL), permettant d’afficher :

- Le document original (PDF ou image)
- Les données extraites automatiquement
- Le score global de risque et les sous-scores
- Les signaux de fraude et facteurs explicatifs

Cette vue est consultative uniquement (aucune modification ni décision dans ce ticket) et s’intègre dans la fonctionnalité « Interface de revue analyste ».

---

## 2. Périmètre fonctionnel

La vue de visualisation couvre :

- L’affichage sécurisé du document source
- La présentation structurée des données extraites
- L’affichage du score global (0–100)
- L’affichage des sous-scores par catégorie
- La liste des signaux de fraude déclenchés
- Les informations de traçabilité (versions modèles et règles)

Ne sont pas inclus dans ce ticket :

- La correction des champs
- La validation / rejet
- L’ajout de commentaires

---

## 3. Accès à la vue

Conditions d’accès :

- Utilisateur authentifié.
- Appartenance au même `tenant_id` que le document.
- Rôle autorisé : analyste, superviseur, administrateur client, administrateur plateforme, auditeur (lecture seule).

L’accès se fait depuis la liste des documents à revoir via l’identifiant du document.

---

## 4. Structure de l’interface

### 4.1 Zone de visualisation du document

Fonctionnalités :

- Affichage du document original (PDF natif, PDF scanné ou image JPG/PNG).
- Navigation multi-pages (si applicable).
- Zoom avant / arrière.
- Défilement fluide.

Règles de gestion :

- Le document affiché correspond strictement au fichier stocké lors de l’upload.
- Aucun élément du document ne peut être modifié.
- Si le document n’est plus disponible (rétention expirée), un message explicite est affiché.

---

### 4.2 Panneau des données extraites

Affichage structuré et dynamique selon le type de document :

- Facture : numéro, date, montant, devise, fournisseur, client, IBAN, etc.
- Pièce d’identité : nom, prénom, date de naissance, numéro de document, date d’expiration, etc.
- Relevé bancaire : titulaire, IBAN, période, transactions clés, etc.

Pour chaque champ affiché :

- Valeur actuelle enregistrée.
- Niveau de confiance associé.

Règles de gestion :

- Les champs à faible confiance peuvent être visuellement distingués.
- Les données affichées correspondent à la version persistée la plus récente.
- Les valeurs originales ne sont pas supprimées mais ne sont pas modifiables depuis cette vue.

---

### 4.3 Panneau des scores

Affichage :

- Score global de risque (0–100).
- Sous-scores par catégorie :
  - Intégrité documentaire
  - Cohérence métier
  - Identité
  - Anomalies visuelles
  - Suspicion de génération/modification par IA

Règles de gestion :

- Le score est présenté comme indicateur de risque uniquement.
- Les valeurs affichées correspondent aux résultats persistés du pipeline.
- Les seuils configurés pour le tenant ne sont pas modifiables depuis cette vue.

---

### 4.4 Panneau des signaux et explicabilité

Affichage :

- Liste des règles déclenchées.
- Anomalies détectées.
- Facteurs ayant contribué au score.
- Niveau de confiance global.
- Version des modèles et des règles appliquées.

Objectif :

- Permettre à l’analyste de comprendre l’origine du score.
- Garantir la transparence et l’auditabilité.

Les informations doivent être présentées de manière lisible et structurée.

---

## 5. Contraintes UX

- Interface responsive (desktop prioritaire).
- Chargement progressif si nécessaire (document, puis données, puis scores).
- Gestion des états : chargement, erreur, document introuvable, accès refusé.

---

## 6. Dépendances fonctionnelles

- Module Authentification & RBAC
- Module Extraction structurée
- Module Moteur de scoring
- Module Audit & traçabilité
- Module Stockage documentaire

La vue consomme exclusivement les données persistées par le pipeline asynchrone.