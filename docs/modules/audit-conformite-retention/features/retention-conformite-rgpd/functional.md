## Fonctionnalité : Rétention & Conformité RGPD

### 1. Objectif métier
Permettre à chaque tenant de configurer et appliquer une politique de rétention des données conforme au RGPD, incluant la conservation limitée des documents sources, des résultats structurés et des logs d’audit, ainsi que la suppression automatique ou à la demande.

Cette fonctionnalité garantit :
- La limitation de la durée de conservation des données.
- La possibilité d’exercer le droit à l’effacement.
- La traçabilité complète des opérations de suppression.
- La sécurisation des données via chiffrement.

---

### 2. Périmètre

La fonctionnalité couvre :
- Configuration des durées de rétention par tenant.
- Application automatique des règles de conservation.
- Suppression automatique planifiée (purge).
- Suppression manuelle (à la demande) d’un document ou d’un ensemble de données.
- Gestion des états du cycle de vie des documents.
- Journalisation systématique des opérations liées à la rétention.

Ne couvre pas :
- La définition des règles de scoring ou d’audit (gérées par d’autres sous-modules).

---

### 3. Règles de gestion

#### 3.1 Configuration de la rétention
Pour chaque tenant, il doit être possible de configurer :
- Durée de conservation des documents sources (en jours).
- Durée de conservation des résultats structurés.
- Durée de conservation des logs d’audit.

Les règles suivantes s’appliquent :
- Toute modification de politique est historisée.
- Une nouvelle politique ne s’applique qu’aux données non encore supprimées.
- Les durées doivent être des valeurs positives.

#### 3.2 Cycle de vie d’un document
États possibles :
- Actif
- Archivé
- En attente de suppression
- Supprimé

Un document passe automatiquement en “En attente de suppression” lorsque la durée configurée est atteinte.

#### 3.3 Suppression automatique
- Un mécanisme planifié identifie les données arrivées à expiration.
- Les documents sources sont supprimés du stockage objet.
- Les résultats structurés peuvent être supprimés ou anonymisés selon la politique.
- Chaque suppression doit générer un événement d’audit.

#### 3.4 Suppression à la demande (droit à l’effacement)
- Un administrateur autorisé peut déclencher la suppression d’un document.
- La suppression doit inclure les données associées (résultats, métadonnées métier), sous réserve des obligations contractuelles d’audit.
- Les logs d’audit peuvent être conservés si requis, mais doivent être minimisés.
- L’action est journalisée avec l’identité du demandeur.

#### 3.5 Sécurité & chiffrement
- Les documents sont chiffrés au repos.
- Les données sont chiffrées en transit.
- La suppression doit garantir l’inaccessibilité définitive du document source.

---

### 4. Critères d’acceptation

- Un tenant peut configurer ses durées de rétention via interface ou API.
- Les documents expirés sont automatiquement identifiés et traités.
- Les fichiers supprimés ne sont plus accessibles via API ni interface.
- Toute suppression génère une entrée d’audit consultable.
- Les données d’un tenant ne peuvent être impactées par la politique d’un autre.
- Le système respecte l’isolation multi-tenant et le RBAC.

---

### 5. Dépendances
- Module Authentification & RBAC (contrôle d’accès).
- Module Stockage S3 (suppression physique).
- Module Audit (journalisation obligatoire).
- Module Pipeline (mise à jour des statuts).

La fonctionnalité est transverse et obligatoire pour la conformité réglementaire globale.