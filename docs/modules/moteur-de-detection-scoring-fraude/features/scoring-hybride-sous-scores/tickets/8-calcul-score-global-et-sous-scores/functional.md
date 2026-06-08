## Ticket : Calcul score global et sous-scores

### 1. Objectif
Implémenter l’algorithme de scoring hybride permettant de :

- Calculer les **5 sous-scores obligatoires (0–100)**.
- Calculer le **score global de risque (0–100)**.
- Générer et stocker les **facteurs explicatifs détaillés** associés aux scores.
- Persister l’ensemble des résultats pour audit, API et Human-in-the-Loop.

Ce ticket concrétise la logique centrale d’agrégation du module *Scoring hybride & sous-scores*.

---

### 2. Périmètre fonctionnel

Le ticket couvre :

1. Agrégation des signaux issus :
   - Du Rule Engine
   - Du AI Scoring Engine

2. Calcul des sous-scores suivants :
   - Intégrité documentaire
   - Cohérence métier
   - Identité
   - Anomalies visuelles
   - Suspicion IA / génération artificielle

3. Calcul du score global :
   - Agrégation pondérée des sous-scores
   - Application des pondérations spécifiques au tenant
   - Normalisation sur échelle 0–100

4. Détermination des éléments associés :
   - Niveau de confiance
   - Indication de traitement (validation automatique possible / revue humaine requise)

5. Construction et stockage des facteurs explicatifs :
   - Règles déclenchées
   - Signaux IA activés
   - Contribution relative de chaque signal
   - Pondérations appliquées
   - Versions modèles et règles

---

### 3. Règles fonctionnelles

- Tous les scores (global et sous-scores) sont compris entre 0 et 100.
- Un score élevé indique un risque plus élevé.
- Le calcul est déterministe pour une même combinaison :
  - Données d’entrée
  - Versions modèles
  - Versions règles
  - Configuration tenant
- Les pondérations sont isolées par `tenant_id`.
- Les modifications de configuration ne sont pas rétroactives.
- L’absence d’un type de signal ne bloque pas le calcul :
  - Le système applique une dégradation contrôlée.
  - L’absence est documentée dans les facteurs explicatifs.
- Aucun score n’implique une décision binaire de fraude.

---

### 4. Résultat attendu

À l’issue de l’exécution :

- Un enregistrement `fraud_analysis` est créé.
- 5 enregistrements `fraud_subscore` sont créés.
- Les signaux détaillés sont persistés dans `fraud_signal`.
- Les informations sont exploitables par :
  - API REST
  - Interface Human-in-the-Loop
  - Module Audit
  - Tableau de bord

Le moteur doit fonctionner de manière stateless et indépendante pour chaque document.