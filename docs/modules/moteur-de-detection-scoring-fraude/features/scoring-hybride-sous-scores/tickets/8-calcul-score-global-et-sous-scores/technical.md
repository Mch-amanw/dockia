## Implémentation technique – Calcul score global et sous-scores

### 1. Position dans le pipeline

Le calcul est exécuté dans le composant **Aggregation Engine**, après réception :

- `rule_signals` (scores partiels + métadonnées)
- `ai_signals` (probabilités + indicateurs)
- `tenant_configuration`

Entrées principales :
- job_id
- tenant_id
- rule_results
- ai_results
- model_version
- rules_version

---

### 2. Étapes d’implémentation

#### 2.1 Mapping des signaux

- Chaque signal est associé à une catégorie de sous-score.
- Les signaux incluent :
  - type (`rule` | `ai`)
  - code
  - raw_score
  - metadata

Un mapping interne catégorise les signaux vers :
- integrity
- business_coherence
- identity
- visual_anomalies
- ai_suspicion

---

#### 2.2 Normalisation

- Conversion de tous les `raw_score` vers une échelle 0–100.
- Application de bornes strictes [0, 100].
- Gestion des valeurs manquantes.

La normalisation doit être purement déterministe.

---

#### 2.3 Calcul des sous-scores

Pour chaque catégorie :

- Agrégation des signaux associés.
- Calcul d’un score consolidé.
- Application éventuelle d’une pondération interne si définie.

Sortie :
```
{
  category: string,
  score: float,
  weight: float
}
```

---

#### 2.4 Application des pondérations tenant

- Chargement dynamique via module Configuration Tenant.
- Pondérations appliquées aux sous-scores.
- Fallback vers configuration par défaut si absente.

Formule conceptuelle :

```
global_score = sum(subscore_i * weight_i) / sum(weight_i)
```

Le résultat est normalisé entre 0 et 100.

---

#### 2.5 Niveau de confiance

Calculé à partir :
- Nombre de signaux actifs
- Complétude des catégories
- Disponibilité des modèles IA

Stocké dans `fraud_analysis.confidence_level`.

---

#### 2.6 Indication de traitement

Comparaison du `global_score` au seuil configuré :

- score < threshold → `auto_validation_possible`
- score ≥ threshold → `human_review_required`

Le seuil est chargé dynamiquement par `tenant_id`.

---

### 3. Persistance

#### 3.1 Table `fraud_analysis`

Insertion :
- id
- job_id
- tenant_id
- global_score
- confidence_level
- model_version
- rules_version
- created_at

---

#### 3.2 Table `fraud_subscore`

Insertion de 5 lignes minimum :
- analysis_id
- tenant_id
- category
- score
- weight

---

#### 3.3 Table `fraud_signal`

Insertion d’un enregistrement par signal :
- analysis_id
- tenant_id
- type
- code
- description
- contribution

La contribution correspond à l’impact relatif du signal dans le calcul.

---

### 4. Gestion des cas dégradés

- Si un moteur IA est indisponible :
  - Les signaux manquants sont exclus du calcul.
  - Une entrée explicite est ajoutée dans `fraud_signal`.

- Si une catégorie ne contient aucun signal :
  - Score neutre cohérent avec absence d’augmentation de risque.
  - La situation est tracée.

Le moteur ne doit jamais lever d’erreur bloquante si des signaux partiels sont disponibles.

---

### 5. Contraintes techniques

- Implémentation stateless.
- Aucun accès inter-tenant.
- Journalisation structurée des étapes de calcul.
- Déterminisme garanti.
- Compatible avec exécution < 30 secondes dans pipeline global.
- Extensible pour ajout futur de nouvelles catégories.

---

### 6. Dépendances

Dépend de :
- Rule Engine
- AI Scoring Engine
- Module Configuration Tenant
- Module Audit & Historisation

Expose :
- Résultat structuré pour API REST
- Données explicatives pour Human-in-the-Loop
- Indicateurs pour Tableau de bord