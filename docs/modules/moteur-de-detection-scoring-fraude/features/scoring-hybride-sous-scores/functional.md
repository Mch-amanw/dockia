## Fonctionnalité : Scoring hybride & sous-scores

### 1. Objectif métier
La fonctionnalité **Scoring hybride & sous-scores** a pour objectif de produire une évaluation synthétique et explicable du risque associé à un document analysé.

Elle calcule :
- Un **score global de risque (0–100)**.
- Des **sous-scores détaillés par catégorie**.
- Les **facteurs explicatifs associés** à ces scores.

Cette évaluation alimente :
- La décision automatique (selon seuil configuré par tenant).
- La priorisation des dossiers en Human-in-the-Loop.
- Les exports API pour les systèmes clients.

⚠️ Aucun score ne constitue une décision binaire de fraude. Il s’agit exclusivement d’un indicateur probabiliste.

---

### 2. Périmètre
La fonctionnalité couvre :
- L’agrégation des signaux issus du Rule Engine et de l’AI Scoring Engine.
- La normalisation des scores partiels.
- Le calcul des sous-scores obligatoires (MVP).
- Le calcul du score global pondéré.
- La génération des éléments d’explicabilité associés.

Hors périmètre :
- L’implémentation des règles métier elles-mêmes.
- L’entraînement ou la définition interne des modèles IA.
- La configuration des seuils (gérée par le module Configuration Tenant).

---

### 3. Sous-scores obligatoires (MVP)

Chaque document analysé doit produire les sous-scores suivants (0–100) :

1. **Intégrité documentaire**  
2. **Cohérence métier**  
3. **Identité**  
4. **Anomalies visuelles**  
5. **Suspicion IA / génération artificielle**  

Chaque sous-score :
- Est calculé à partir des signaux pertinents.
- Est normalisé sur une échelle commune (0–100).
- Dispose d’une pondération configurable par tenant.

---

### 4. Calcul du score global

Le score global :
- Est une agrégation pondérée des sous-scores.
- Utilise les pondérations définies dans la configuration du tenant.
- Est normalisé entre 0 et 100.

Le moteur doit également produire :
- Un **niveau de confiance**.
- Une indication de traitement :
  - "Validation automatique possible" (si score < seuil configuré)
  - "Revue humaine requise" (si score ≥ seuil)

Les modifications de pondération ne sont pas rétroactives.

---

### 5. Règles de gestion

- Tous les scores sont compris entre 0 et 100.
- Un score élevé indique un risque plus élevé.
- L’absence d’un type de signal ne bloque pas le calcul (dégradation contrôlée).
- Le calcul doit être déterministe pour une même version de modèles, règles et configuration.
- Les pondérations sont isolées par tenant.

---

### 6. Explicabilité

Pour chaque score (global et sous-scores), la fonctionnalité doit fournir :

- Liste des règles déclenchées.
- Signaux IA activés.
- Contribution relative de chaque signal.
- Pondérations appliquées.
- Version des modèles et des règles utilisées.

Ces éléments doivent être exploitables par :
- L’API REST.
- L’interface Human-in-the-Loop.
- Le module d’audit.

---

### 7. Critères d’acceptation

- ✅ Un document traité retourne systématiquement un score global et 5 sous-scores.
- ✅ Les scores sont compris entre 0 et 100.
- ✅ Les pondérations spécifiques au tenant sont appliquées correctement.
- ✅ Les facteurs explicatifs sont présents et cohérents avec les signaux détectés.
- ✅ Deux exécutions avec mêmes données et mêmes versions produisent le même résultat.
- ✅ L’absence d’un sous-module IA n’empêche pas la production d’un score (avec signalement dans l’explicabilité).