Spécification Technique

## 1. Architecture générale
Architecture cloud-native conteneurisée.

Composants principaux :
- API Backend (FastAPI – Python)
- Service de traitement asynchrone (workers)
- Base de données PostgreSQL
- Stockage objet compatible S3
- Moteur OCR
- Services IA (extraction + détection fraude)

Architecture orientée microservices ou services modulaires.

---

## 2. Multi-tenancy
- Base de données unique.
- Isolation logique via `tenant_id`.
- Conception évolutive vers schéma ou base dédiée.

---

## 3. Pipeline asynchrone
Flux technique :
1. Upload → création d’un enregistrement Job
2. Mise en file d’attente
3. Worker traite le document
4. Stockage résultats
5. Mise à jour statut
6. Notification webhook

Objectifs :
- Scalabilité horizontale
- Résilience aux pics de charge
- Traitement < 30 secondes pour la majorité des documents

---

## 4. Stockage
- Documents : stockage objet S3 compatible
- Chiffrement au repos
- TLS en transit
- Séparation logique par tenant
- Politique de rétention configurable

---

## 5. Moteur IA

### 5.1 OCR
- PaddleOCR, Tesseract ou équivalent

### 5.2 Extraction
- Modèles IA pré-entraînés
- Structuration des champs par type de document

### 5.3 Détection de fraude
- Système hybride : règles + modèles IA
- Détection probabiliste
- Analyse forensique avancée

### 5.4 Versioning
- Version des modèles enregistrée
- Historisation des règles appliquées

---

## 6. API REST
Fonctionnalités :
- Upload document
- Récupération statut job
- Récupération résultats
- Configuration client
- Webhooks

Authentification :
- OAuth2 / OIDC
- JWT pour accès API
- Compatibilité SAML et OIDC pour SSO entreprise

---

## 7. Sécurité
- Conformité RGPD
- Hébergement en Europe
- Chiffrement TLS
- Chiffrement au repos
- Journalisation complète des accès
- Séparation stricte dev / staging / production
- Gestion fine des permissions (RBAC)

---

## 8. Audit et traçabilité
Stockage des éléments suivants :
- Entrées brutes
- Résultats OCR
- Scores détaillés
- Règles appliquées
- Version des modèles
- Historique des actions utilisateurs

Objectif : auditabilité complète.

---

## 9. Scalabilité
- Conteneurisation Docker
- Scalabilité horizontale des workers
- Architecture compatible montée en charge automatique
- Support de plusieurs milliers de documents/jour/client

---