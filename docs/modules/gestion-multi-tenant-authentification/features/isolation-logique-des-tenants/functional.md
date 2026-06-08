## Fonctionnalité : Isolation logique des tenants

### 1. Objectif métier
Garantir une séparation stricte et systématique des données entre les différents clients (tenants) de Dockia afin d’assurer :
- La confidentialité absolue des données.
- La conformité réglementaire (RGPD, exigences bancaires).
- La confiance des clients grands comptes.

Aucune donnée d’un tenant ne doit être accessible, consultable ou modifiable par un autre tenant.

---

### 2. Périmètre
Cette fonctionnalité couvre :
- L’association obligatoire d’un `tenant_id` à toutes les entités métier.
- Le filtrage automatique des accès aux données via le contexte d’authentification.
- La protection contre tout accès inter-tenant via API.
- La séparation logique dans le stockage des documents (S3).

Hors périmètre (MVP) :
- Isolation physique par base ou schéma dédié.
- Déploiement d’instances dédiées par client.

---

### 3. Règles de gestion

#### 3.1 Association obligatoire au tenant
- Toute entité métier (document, job, résultat, règle, configuration, audit log, utilisateur) doit être liée à un `tenant_id`.
- Aucun enregistrement métier ne peut être créé sans `tenant_id` valide.
- Le `tenant_id` est dérivé du contexte d’authentification et ne peut pas être fourni librement par l’utilisateur via l’API.

#### 3.2 Accès aux données
- Un utilisateur ne peut accéder qu’aux ressources associées à son `tenant_id`.
- Toute tentative d’accès à une ressource appartenant à un autre tenant doit :
  - Retourner une erreur d’autorisation (403 ou 404 selon stratégie retenue).
  - Être journalisée dans l’audit.

#### 3.3 Administrateur plateforme
- L’administrateur plateforme peut accéder aux données de plusieurs tenants uniquement dans le cadre des opérations autorisées (support, supervision).
- Ces accès doivent être strictement journalisés.

#### 3.4 Isolation des configurations
- Les règles de scoring, seuils, webhooks et politiques de rétention sont strictement isolés par tenant.
- Aucun paramètre d’un tenant ne peut être visible ou réutilisable par un autre.

#### 3.5 Isolation du stockage documentaire
- Les documents stockés sur S3 doivent être organisés de manière logique par tenant.
- Un utilisateur ne peut accéder qu’aux fichiers correspondant à son `tenant_id`.

---

### 4. Critères d’acceptation
- ✅ Toute création d’entité métier inclut automatiquement un `tenant_id` valide.
- ✅ Les endpoints API retournent uniquement les données du tenant courant.
- ✅ Une tentative d’accès à une ressource d’un autre tenant est bloquée.
- ✅ Les logs d’audit enregistrent les tentatives d’accès non autorisées.
- ✅ Les documents S3 sont stockés dans un espace logique séparé par tenant.
- ✅ Les tests automatisés couvrent les cas d’accès inter-tenant interdits.

---

### 5. Risques couverts
- Fuite de données entre clients.
- Mauvaise configuration d’API exposant des données inter-tenant.
- Erreur humaine lors de requêtes base de données non filtrées.

Cette fonctionnalité est critique pour la sécurité et constitue un prérequis à toute mise en production.