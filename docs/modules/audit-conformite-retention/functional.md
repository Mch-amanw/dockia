## Module : Audit, Conformité & Rétention

### 1. Rôle du module
Le module **Audit, Conformité & Rétention** garantit l’auditabilité complète, la traçabilité des décisions, la conformité RGPD et la gestion du cycle de vie des données dans Dockia.

Il couvre :
- La journalisation exhaustive des traitements et actions utilisateurs.
- Le versioning des modèles IA et des règles métier.
- La gestion des politiques de rétention configurables par tenant.
- Les mécanismes de suppression et de purge automatique.

Ce module est transverse à l’ensemble de la plateforme (API, pipeline IA, Human-in-the-Loop, stockage).

---

### 2. Journalisation des traitements

#### 2.1 Événements journalisés
Le système doit enregistrer au minimum :
- Upload du document (tenant, utilisateur, horodatage).
- Création et transitions de statut du job.
- Résultats OCR.
- Données extraites.
- Scores globaux et sous-scores.
- Règles appliquées lors du scoring.
- Version des modèles IA utilisés.
- Actions Human-in-the-Loop (correction, validation, rejet, commentaire).
- Accès aux résultats (consultation, export).
- Événements de suppression/purge.

#### 2.2 Exigences fonctionnelles
- Chaque événement doit être horodaté.
- Chaque événement doit être lié à un `tenant_id`.
- Les actions humaines doivent être associées à un `user_id` et un rôle.
- Les logs doivent être non modifiables (append-only logique).
- Les logs doivent être consultables par rôle autorisé (auditeur, administrateur client, administrateur plateforme).

---

### 3. Versioning des modèles et règles

#### 3.1 Modèles IA
Pour chaque traitement :
- Identifiant du modèle utilisé.
- Version du modèle.
- Type de modèle (OCR, extraction, scoring, détection fraude).

Le système doit permettre :
- L’enregistrement d’une nouvelle version de modèle.
- L’association automatique de la version active lors d’un traitement.
- La conservation de l’historique des versions.

#### 3.2 Règles métier
- Historisation des règles appliquées au moment du scoring.
- Conservation de la configuration des seuils par tenant.
- Traçabilité des modifications de règles (qui, quand, quoi).

---

### 4. Conformité RGPD

#### 4.1 Principes couverts
- Minimisation des données.
- Droit à l’effacement.
- Limitation de conservation.
- Traçabilité des accès.

#### 4.2 Fonctionnalités
- Possibilité de suppression d’un document et de ses données associées (logique ou physique selon configuration).
- Journalisation de toute suppression.
- Capacité à répondre à une demande d’effacement pour un tenant.
- Conservation des éléments strictement nécessaires à l’audit, même après suppression du document source, si contractuellement requis.

---

### 5. Politique de rétention

#### 5.1 Configuration par tenant
Chaque tenant peut configurer :
- Durée de conservation des documents sources.
- Durée de conservation des résultats structurés.
- Durée de conservation des logs d’audit.

#### 5.2 Cycle de vie
États possibles :
- Actif
- Archivé
- En attente de suppression
- Supprimé

#### 5.3 Suppression automatique
- Exécution périodique d’un mécanisme de purge.
- Suppression conforme à la politique définie.
- Journalisation obligatoire de chaque purge.

---

### 6. Consultation et export d’audit

Fonctionnalités :
- Consultation des logs par document.
- Consultation des logs par période.
- Filtres par type d’événement, utilisateur, statut.
- Export des logs (format structuré exploitable).

Accès restreint selon RBAC.

---

### 7. Règles de gestion
- Aucune donnée ne peut être modifiée sans traçabilité.
- Les logs d’audit ne peuvent être supprimés sans autorisation spécifique.
- Les données d’un tenant ne sont jamais accessibles à un autre.
- Les suppressions doivent respecter la politique de rétention configurée.
- Toute décision humaine doit être historisée et conservée.

---

### 8. Dépendances
- Module Authentification & RBAC.
- Module Pipeline de traitement.
- Module Moteur IA.
- Module Stockage S3.
- Module Human-in-the-Loop.

Le module est transverse et requis pour conformité et auditabilité globale.