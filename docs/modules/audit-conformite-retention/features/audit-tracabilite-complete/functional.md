## Fonctionnalité : Audit & traçabilité complète

### 1. Objectif métier
Garantir une traçabilité exhaustive, fiable et exploitable de l’ensemble des traitements documentaires et des actions humaines au sein de Dockia.

Cette fonctionnalité permet :
- D’assurer l’auditabilité complète des décisions (automatiques et humaines).
- De répondre aux exigences réglementaires (RGPD, conformité bancaire, assurance).
- De fournir une visibilité détaillée aux administrateurs et auditeurs.

---

### 2. Périmètre fonctionnel
La fonctionnalité couvre l’historisation des éléments suivants :

#### 2.1 Événements techniques
- Upload d’un document.
- Création d’un job.
- Transitions de statut du job (pending, processing, completed, failed, etc.).
- Résultats OCR.
- Données extraites.
- Calcul du score global et des sous-scores.
- Règles appliquées lors du scoring.
- Versions des modèles IA utilisés (OCR, extraction, fraude).
- Notification webhook envoyée.

#### 2.2 Actions humaines (Human-in-the-Loop)
- Correction de champs extraits.
- Validation ou rejet d’un document.
- Ajout de commentaires.
- Consultation ou export des résultats.

#### 2.3 Métadonnées obligatoires
Chaque événement doit inclure :
- `tenant_id`
- Identifiant du document et/ou du job (si applicable)
- `user_id` (si action humaine)
- Rôle de l’utilisateur (si applicable)
- Horodatage précis
- Type d’événement
- Payload descriptif structuré

---

### 3. Règles de gestion
- Les logs sont logiquement append-only : aucune modification ou suppression métier autorisée.
- Toute action humaine doit être tracée de manière explicite.
- Chaque traitement doit être lié à la version exacte des modèles et des règles utilisées.
- Les données d’un tenant ne sont visibles que par les utilisateurs autorisés de ce tenant.
- Les suppressions (manuelles ou automatiques) doivent elles-mêmes être journalisées.
- L’audit doit permettre de reconstituer intégralement la chaîne de décision d’un document.

---

### 4. Consultation et exploitation
Les utilisateurs autorisés (RBAC) doivent pouvoir :
- Consulter l’historique complet d’un document.
- Consulter l’historique d’un job.
- Filtrer les logs par période, type d’événement, utilisateur.
- Exporter les logs dans un format structuré exploitable.

---

### 5. Critères d’acceptation
- ✅ Chaque étape du pipeline génère une entrée d’audit.
- ✅ Les actions Human-in-the-Loop apparaissent avec utilisateur et rôle.
- ✅ Les versions des modèles sont systématiquement associées aux traitements.
- ✅ Les logs sont isolés par tenant.
- ✅ Un auditeur peut reconstituer l’ensemble des décisions prises sur un document.
- ✅ Les performances globales du traitement ne sont pas dégradées de manière significative.

---

### 6. Hors périmètre
- Modification ou suppression rétroactive des logs.
- Analyse avancée des logs (BI, dashboards complexes) au-delà des filtres standards.

Cette fonctionnalité constitue le socle d’auditabilité réglementaire et de confiance de la plateforme.