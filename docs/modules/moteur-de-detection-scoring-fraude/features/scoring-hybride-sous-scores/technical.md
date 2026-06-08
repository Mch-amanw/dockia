## Spécification technique – Scoring hybride & sous-scores

### 1. Positionnement technique

La fonctionnalité est implémentée principalement dans le composant **Aggregation Engine**, en interaction avec :
- Rule Engine
- AI Scoring Engine
- Explainability Builder
- Module Configuration Tenant

Elle est exécutée dans le pipeline asynchrone après réception de tous les signaux partiels.

---

### 2. Flux technique

1. Récupération des signaux :
   - Résultats règles (scores partiels + métadonnées)
   - Résultats IA (probabilités, scores)

2. Mapping des signaux vers catégories de sous-scores.

3. Normalisation :
   - Conversion vers échelle commune 0–100.
   - Gestion des bornes et valeurs manquantes.

4. Application des pondérations (chargées dynamiquement selon `tenant_id`).

5. Calcul :
   - Sous-scores par catégorie
   - Score global agrégé

6. Transmission des données au Explainability Builder.

---

### 3. Modèle de données impacté

Tables concernées :

- `fraud_analysis`
  - global_score
  - confidence_level
  - model_version
  - rules_version

- `fraud_subscore`
  - category
  - score
  - weight

- `fraud_signal`
  - type (rule | ai)
  - code
  - description
  - contribution

Chaque enregistrement inclut `tenant_id`.

---

### 4. Configuration dynamique

Les pondérations et seuils sont récupérés via le module Configuration Tenant :
- Chargement par `tenant_id`.
- Fallback vers configuration par défaut si absente.
- Non mise en cache persistante sans invalidation contrôlée.

---

### 5. Gestion des cas dégradés

- Si un modèle IA est indisponible :
  - La catégorie concernée est calculée avec les signaux disponibles.
  - L’indisponibilité est consignée dans les signaux.

- Si aucun signal pour une catégorie :
  - Score neutre défini par règle interne cohérente (ex. absence d’augmentation du risque).

Le moteur reste stateless.

---

### 6. Contraintes

- Temps d’exécution compatible avec pipeline < 30 secondes globalement.
- Calcul purement déterministe.
- Aucune dépendance inter-document.
- Extensibilité : possibilité d’ajouter de nouvelles catégories de sous-scores sans refonte majeure.

---

### 7. Sécurité & audit

- Journalisation des étapes de calcul.
- Persistance des pondérations appliquées.
- Persistance des contributions individuelles.
- Isolation stricte via `tenant_id`.

---

### 8. Dépendances

Dépend :
- Rule Engine
- AI Scoring Engine
- Configuration Tenant
- Module Audit & Historisation

Expose :
- Résultats structurés à l’API REST
- Données explicatives au module Human-in-the-Loop
- Indicateurs au Tableau de bord