# Module : Moteur de Détection & Scoring Fraude

## 1. Rôle du module dans Dockia
Le module **Moteur de Détection & Scoring Fraude** est responsable de l’analyse des documents traités (après OCR et extraction structurée) afin d’évaluer le niveau de risque de fraude.

Il produit :
- Un **score global de risque (0–100)**.
- Des **sous-scores par catégorie**.
- Une liste de **signaux et facteurs explicatifs**.

Ce module constitue le cœur décisionnel de Dockia et alimente :
- L’API REST (résultats exploitables par les clients).
- L’interface Human-in-the-Loop.
- Le système d’audit et de traçabilité.

⚠️ Règle clé : le module ne déclare jamais un document comme frauduleux avec certitude. Il produit uniquement un **score de risque probabiliste**.

---

## 2. Entrées du module
Le moteur reçoit :

- Données structurées extraites (champs normalisés selon type de document).
- Résultats OCR (texte brut + métadonnées éventuelles).
- Métadonnées du document (format, taille, hash, date upload, etc.).
- Identifiant du tenant.
- Configuration du tenant (règles actives, seuils, pondérations).
- Version des modèles IA actifs.

---

## 3. Fonctionnalités principales

### 3.1 Scoring hybride
Le moteur combine :

1. **Règles métier déterministes**  
   - Cohérence des montants (factures)
   - Concordance identité (nom, date de naissance, numéro ID)
   - Détection d’incohérences structurelles
   - Vérifications métier spécifiques au type de document

2. **Modèles IA probabilistes**  
   - Détection d’anomalies statistiques
   - Suspicion de génération ou modification par IA
   - Détection d’altérations visuelles
   - Analyse comportementale ou structurelle

3. **Détection d’anomalies transverses**  
   - Incohérences entre champs
   - Divergence entre texte OCR et structure PDF
   - Anomalies de métadonnées

Le résultat est une agrégation pondérée configurable.

---

### 3.2 Production de sous-scores
Sous-scores obligatoires (MVP) :

- Intégrité documentaire
- Cohérence métier
- Identité
- Anomalies visuelles
- Suspicion IA / génération artificielle

Chaque sous-score est normalisé entre 0 et 100.

---

### 3.3 Calcul du score global
Le score global :
- Est calculé à partir des sous-scores.
- Utilise une pondération configurable par tenant.
- Produit une valeur 0–100.

Le moteur retourne également :
- Niveau de confiance.
- Seuil automatique (si configuré).
- Indication :
  - Validation automatique possible
  - Revue humaine requise

---

### 3.4 Explicabilité
Pour chaque score, le moteur doit fournir :

- Liste des règles déclenchées
- Signaux IA activés
- Contribution relative de chaque facteur
- Version des modèles utilisés
- Règles et pondérations appliquées

Ces éléments sont obligatoires pour audit et affichage dans le module Human-in-the-Loop.

---

### 3.5 Configuration par tenant
Le module supporte :

- Activation/désactivation de catégories de règles
- Pondérations personnalisées
- Seuil de validation automatique
- Paramètres de sensibilité

Aucune configuration d’un tenant ne doit impacter un autre tenant.

---

### 3.6 Interaction avec Human-in-the-Loop
Le moteur :

- Transmet tous les signaux explicatifs à l’interface analyste.
- Enregistre les décisions humaines.
- Permet l’exploitation ultérieure des validations pour amélioration continue.

---

## 4. Règles de gestion

- Aucun score ne doit être binaire (fraude / non fraude).
- Tous les scores doivent être traçables.
- Toute décision doit être reproductible à partir des données historisées.
- Les règles appliquées doivent être versionnées.
- Les modifications de configuration ne sont pas rétroactives.

---