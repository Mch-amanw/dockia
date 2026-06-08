Spécification Technique (Version enrichie avec stack cible)

## 1. Architecture générale
Architecture cloud-native conteneurisée, pensée pour Kubernetes.

Composants principaux :
- Backend API (FastAPI – Python 3.12+)
- Workers asynchrones (Celery)
- Base de données PostgreSQL
- Cache & broker Redis
- Stockage objet S3 compatible
- Services IA (OCR, extraction, fraude)
- Frontend Next.js (React + TypeScript)

Cloud cible MVP :
- AWS (prioritaire)
- Alternatives possibles : OVHcloud, Scaleway

---

## 2. Stack Backend
- Python 3.12+
- FastAPI
- Pydantic
- SQLAlchemy
- Alembic (migrations)
- PostgreSQL
- Redis
- Celery (traitement asynchrone)

Architecture modulaire compatible microservices.

---

## 3. Frontend
- Next.js (React + TypeScript)
- Tailwind CSS
- ShadCN UI
- React Query (TanStack Query)
- Authentification via OIDC

Fonctionnalités couvertes :
- Gestion documentaire
- Visualisation des documents
- Revue des résultats IA
- Dashboard
- Administration multi-tenant

---

## 4. Orchestration et déploiement
### Développement
- Docker obligatoire
- Docker Compose pour environnement local

### Production
- Kubernetes
- EKS si AWS retenu

L’architecture doit être compatible Kubernetes dès le MVP.

---

## 5. Stockage
### Documents
- Amazon S3 ou stockage compatible S3
- Chiffrement au repos
- TLS en transit
- Séparation logique par tenant

### Base de données
- PostgreSQL
- Isolation logique via `tenant_id`

### Cache / Broker
- Redis

---

## 6. Pipeline asynchrone
Flux technique :
1. Upload → création Job
2. Mise en file Redis
3. Traitement par worker Celery
4. OCR
5. Extraction
6. Scoring fraude
7. Stockage résultats
8. Notification webhook

Objectifs :
- Scalabilité horizontale des workers
- Résilience aux pics de charge
- Temps moyen de traitement < 30 secondes

---

## 7. IA et OCR
### OCR
- PaddleOCR
- DocTR

### Extraction
- Modèles IA pré-entraînés
- Utilisation possible de LLM pour extraction et validation

### Détection de fraude
- Modèles spécialisés de détection d’anomalies
- Approche hybride (règles + IA)

Architecture compatible avec :
- Intégration future de modèles open source
- Intégration de modèles propriétaires

### Versioning
- Version des modèles enregistrée
- Historisation des règles appliquées

---

## 8. Recherche et filtrage
MVP :
- Recherche via PostgreSQL

Évolution :
- OpenSearch / Elasticsearch

---

## 9. API REST
Fonctionnalités :
- Upload document
- Récupération statut job
- Récupération résultats
- Configuration client
- Webhooks

Authentification :
- OAuth2 / OIDC
- JWT
- Compatibilité SAML et OIDC (SSO entreprise)

---

## 10. Observabilité
- Prometheus (métriques)
- Grafana (visualisation)
- Sentry (gestion erreurs)

Objectif : monitoring applicatif et supervision des traitements.

---

## 11. Sécurité
- Conformité RGPD
- Hébergement Europe
- TLS obligatoire
- Chiffrement au repos
- Journalisation complète des accès
- RBAC
- Séparation dev / staging / production

---

## 12. CI/CD
- GitHub (gestion code source)
- GitHub Actions (CI/CD)
- Build Docker automatisé
- Déploiement automatisé vers staging
- Déploiement automatisé vers production

---

## 13. Scalabilité
- Conteneurisation Docker
- Scalabilité horizontale (API + workers)
- Compatible auto-scaling Kubernetes
- Support de plusieurs milliers de documents/jour/client

---

## 14. Audit et traçabilité
Stockage obligatoire :
- Entrées brutes
- Résultats OCR
- Scores détaillés
- Règles appliquées
- Version des modèles
- Historique des actions utilisateurs

Objectif : auditabilité complète et conformité réglementaire.

---