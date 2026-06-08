# Dockia – Spécification Fonctionnelle (Version enrichie)

## 1. Vision et objectifs
Dockia est une plateforme SaaS multi-clients d’extraction intelligente de documents et de détection de fraude par IA.

Objectifs principaux :
- Extraire automatiquement des données structurées à partir de documents variés.
- Détecter et scorer les risques de fraude.
- Fournir un système d’aide à la décision avec intervention humaine (Human-in-the-Loop).
- Garantir l’auditabilité complète et l’explicabilité des décisions.

---

## 2. Périmètre fonctionnel (MVP)

### 2.1 Types de documents pris en charge
- Factures (fournisseurs et clients)
- Pièces d’identité (CNI, passeports, permis)
- Relevés bancaires

Versions ultérieures :
- Contrats
- Justificatifs administratifs

### 2.2 Formats d’entrée
- PDF natifs
- PDF scannés
- Images JPG, JPEG, PNG

Canaux d’entrée :
- Upload via interface web
- API REST

---

## 3. Fonctionnalités principales

### 3.1 Gestion multi-tenant
- Isolation logique des données via `tenant_id`.
- Configuration spécifique par client (règles, seuils, rétention).
- Administration dédiée par tenant via interface web.

---

### 3.2 Pipeline de traitement documentaire
Flux métier :
1. Upload du document
2. Création d’un job de traitement
3. OCR
4. Extraction des données
5. Analyse anti-fraude
6. Calcul du score
7. Stockage des résultats
8. Notification via webhook ou consultation API

Traitement asynchrone par défaut.

Statuts possibles d’un job :
- `uploaded`
- `processing`
- `completed`
- `failed`
- `under_review`

---

### 3.3 Extraction structurée
- OCR multi-format.
- Extraction des champs clés selon le type de document.
- Restitution au format structuré exploitable via API.
- Affichage structuré dans l’interface analyste.
- Possibilité de correction manuelle par un analyste.

---

### 3.4 Moteur de détection et scoring de fraude
Modèle hybride combinant :
- Règles métier
- Modèles IA pré-entraînés
- Détection d’anomalies

#### Sorties du moteur
- Score global de risque (0–100)
- Sous-scores par catégorie :
  - Intégrité documentaire
  - Cohérence métier
  - Identité
  - Anomalies visuelles
  - Suspicion de génération/modification par IA

Les règles et seuils sont configurables par client.

---

### 3.5 Détection de fraude documentaire
Fraudes ciblées :
- Modification de montants ou données sensibles
- Falsification ou altération
- Incohérences contenu/métadonnées/structure
- Documents générés ou modifiés par IA
- Usurpation d’identité

Approche multicouche :
- Détection probabiliste
- Analyses forensiques avancées (artefacts, compression, couches graphiques)

Les résultats sont présentés comme indicateurs de risque.

---

### 3.6 Human-in-the-Loop
Interface analyste permettant :
- Visualisation du document (viewer intégré)
- Consultation des données extraites
- Visualisation des signaux de fraude
- Consultation du score global et des sous-scores
- Affichage des facteurs explicatifs

Actions possibles :
- Correction des champs
- Validation
- Rejet
- Mise en revue
- Ajout de commentaires

Rôles utilisateurs :
- Administrateur plateforme
- Administrateur client
- Superviseur
- Analyste
- Lecteur / Auditeur

---

### 3.7 Explicabilité et audit
Chaque décision inclut :
- Facteurs influençant le score
- Anomalies détectées
- Niveau de confiance

Historisation obligatoire :
- Version des modèles
- Règles appliquées
- Données d’entrée
- Résultats générés
- Actions et validations humaines

Objectif : auditabilité complète et traçabilité réglementaire.

---

### 3.8 Tableau de bord
Fonctionnalités :
- Suivi des documents traités
- Visualisation des statuts des jobs
- Indicateurs de volume
- Indicateurs de performance
- Historique des décisions
- Filtres par période, statut, type de document

---

### 3.9 Politique de rétention
- Rétention configurable par client
- Conservation des résultats structurés par défaut
- Conservation limitée des documents sources
- Suppression automatique possible
- Respect des exigences RGPD

---

## 4. Règles de gestion clés
- Aucun document n’est considéré frauduleux avec certitude : uniquement score de risque.
- Les seuils de validation automatique sont configurables par client.
- Les décisions humaines sont historisées et exploitables pour amélioration continue.
- Les données d’un tenant ne sont jamais accessibles à un autre.

---

## 5. Interface utilisateur (MVP)
L’interface web (Next.js) doit permettre :
- Upload de documents
- Suivi des traitements
- Revue analyste complète
- Administration multi-tenant
- Gestion des utilisateurs et rôles
- Configuration des seuils de scoring

L’authentification se fait via OIDC (compatible SSO entreprise).

---

# Dockia –