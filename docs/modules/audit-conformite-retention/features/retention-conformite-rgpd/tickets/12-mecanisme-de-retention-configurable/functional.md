## Ticket : Mécanisme de rétention configurable

### 1. Objectif
Permettre à chaque tenant de configurer sa politique de rétention des données (documents sources, résultats structurés, logs d’audit) et garantir l’application automatique de cette politique via un mécanisme de suppression planifiée.

Ce ticket couvre :
- La configuration des durées de conservation par tenant.
- La mise à jour et la consultation de cette configuration.
- L’identification automatique des données expirées.
- Le déclenchement de la suppression automatique conformément à la politique définie.

---

### 2. Périmètre fonctionnel

#### 2.1 Configuration de la politique de rétention
Un administrateur autorisé (administrateur client ou administrateur plateforme) doit pouvoir :
- Consulter la politique de rétention actuelle de son tenant.
- Définir ou modifier :
  - La durée de conservation des documents sources (en jours).
  - La durée de conservation des résultats structurés (en jours).
  - La durée de conservation des logs d’audit (en jours).

Règles :
- Les durées doivent être des entiers positifs.
- Toute modification est journalisée.
- La politique est strictement isolée par `tenant_id`.
- Une modification de politique ne restaure pas des données déjà supprimées.

---

#### 2.2 Application automatique de la rétention
Le système doit appliquer automatiquement la politique configurée :

- Les documents dépassant la durée de conservation définie passent à l’état `pending_deletion`.
- Le document source est supprimé du stockage objet.
- Le statut du document est mis à jour en `deleted`.
- La date effective de suppression est enregistrée.

Pour les résultats structurés :
- Ils sont supprimés ou rendus inaccessibles selon la politique définie.

Pour les logs d’audit :
- Ils sont éligibles à suppression uniquement selon `audit_retention_days`.

Chaque opération de suppression automatique doit générer un événement d’audit.

---

### 3. Règles de gestion

- L’application des règles est strictement isolée par tenant.
- La suppression automatique ne doit pas impacter les documents encore en cours de traitement.
- Le système ne doit jamais rendre accessibles des documents au-delà de la durée configurée.
- Les opérations de purge doivent être traçables.
- En cas d’échec partiel (ex: échec suppression S3), le document ne doit pas être considéré comme supprimé tant que l’opération n’est pas complète.

---

### 4. Acteurs concernés
- Administrateur client : configure la politique de son tenant.
- Administrateur plateforme : supervision et configuration globale si nécessaire.
- Système (Retention Manager) : applique automatiquement les règles.

---

### 5. Dépendances
- Module Authentification & RBAC (contrôle des accès).
- Module Audit (journalisation obligatoire).
- Module Stockage S3 (suppression physique).
- Module Pipeline (gestion des statuts des documents).

Ce ticket constitue la base opérationnelle de la conformité RGPD liée à la limitation de conservation des données.