## Ticket : Journalisation complète des traitements

### 1. Objectif
Mettre en place la journalisation exhaustive et systématique de tous les traitements documentaires et actions utilisateurs afin de garantir une auditabilité complète, conforme aux exigences réglementaires et aux spécifications du module **Audit & traçabilité complète**.

Ce ticket couvre spécifiquement :
- L’enregistrement des versions de modèles IA utilisées.
- L’historisation des règles appliquées lors du scoring.
- La conservation des scores globaux et sous-scores.
- La journalisation des actions utilisateurs (Human-in-the-Loop).

---

### 2. Périmètre fonctionnel

#### 2.1 Journalisation des traitements automatiques
Pour chaque document traité, les éléments suivants doivent être enregistrés :

- Version du modèle OCR utilisé.
- Version du modèle d’extraction utilisé.
- Version du modèle de détection/scoring de fraude utilisé.
- Version des règles métier appliquées.
- Score global de risque (0–100).
- Sous-scores par catégorie (intégrité documentaire, cohérence métier, identité, anomalies visuelles, suspicion IA).
- Résumé des règles déclenchées (identifiants ou codes des règles).

Chaque étape clé du pipeline doit générer un événement distinct (ex: OCR terminé, extraction terminée, scoring terminé).

---

#### 2.2 Journalisation des actions Human-in-the-Loop
Les actions suivantes doivent être tracées :

- Correction d’un champ extrait (ancien valeur / nouvelle valeur).
- Validation d’un document.
- Rejet d’un document.
- Ajout de commentaire.
- Consultation ou export des résultats.

Chaque action doit inclure :
- Identifiant utilisateur.
- Rôle utilisateur.
- Horodatage précis.
- Référence au document et/ou job concerné.

---

#### 2.3 Métadonnées obligatoires
Chaque entrée d’audit doit inclure au minimum :

- `tenant_id`
- `job_id` et/ou `document_id`
- `event_type`
- `created_at`
- `event_payload` structuré
- `model_version` (si applicable)
- `rule_version` (si applicable)
- `user_id` et `user_role` (si action humaine)

---

### 3. Règles de gestion

- Les logs sont en mode append-only logique : aucune modification métier autorisée après insertion.
- Chaque traitement doit être rattaché à la version exacte des modèles et règles utilisés.
- Les actions humaines doivent être traçables individuellement.
- Les données sont strictement isolées par `tenant_id`.
- Toute suppression ultérieure doit elle-même être journalisée.
- L’ensemble des informations journalisées doit permettre de reconstituer intégralement la chaîne de décision d’un document.

---

### 4. Dépendances

- Module Pipeline de traitement (génération des événements techniques).
- Module Moteur IA (fourniture des versions de modèles et scores détaillés).
- Module Human-in-the-Loop (actions utilisateurs).
- Module Authentification & RBAC (identité et rôle utilisateur).
- Module Audit, Conformité & Rétention (stockage et consultation des logs).

Ce ticket constitue un socle critique pour la conformité réglementaire et l’explicabilité des décisions de la plateforme.