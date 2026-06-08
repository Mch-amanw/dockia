## Spécification Technique : Moteur de règles configurables

### 1. Positionnement dans le module
Composant : **Rule Engine** du module Moteur de Détection & Scoring Fraude.

Interaction :
- Reçoit données structurées + métadonnées.
- Retourne signaux + scores partiels à l’Aggregation Engine.

---

### 2. Architecture interne

#### 2.1 Rule Registry
Catalogue des règles disponibles (globales plateforme) :
- rule_code
- document_types
- category (sous-score cible)
- description
- paramètres attendus
- version

Les règles sont implémentées sous forme de classes ou fonctions Python respectant une interface commune :

```
class BaseRule:
    code: str
    category: str
    version: str

    def evaluate(self, document_data, config) -> RuleResult:
        pass
```

---

#### 2.2 Tenant Rule Configuration
Stockée en base (PostgreSQL) :

Table conceptuelle : `tenant_rule_config`
- id
- tenant_id
- rule_code
- enabled (bool)
- parameters (JSONB)
- weight
- severity_level
- rules_version
- updated_at

Chargement dynamique au moment du traitement.

---

#### 2.3 Exécution
Flux :
1. Récupération config tenant.
2. Filtrage règles actives.
3. Instanciation règle.
4. Injection paramètres spécifiques.
5. Évaluation.
6. Production d’un `RuleResult` structuré.

Structure `RuleResult` :
- rule_code
- triggered (bool)
- raw_score
- normalized_score
- contribution
- details (JSON)

---

### 3. Contribution au scoring

- Chaque règle produit un score partiel.
- Les scores sont normalisés (0–100).
- L’Aggregation Engine applique la pondération définie.
- Les contributions sont enregistrées dans `fraud_signal`.

Lien avec :
- `fraud_subscore`
- `fraud_analysis`

---

### 4. Versioning

Deux niveaux de versioning :

1. Version règle (code applicatif).
2. Version configuration tenant.

Lors de l’analyse :
- Snapshot de la configuration utilisé.
- Stockage explicite dans `fraud_analysis.rules_version`.

Garantit reproductibilité.

---

### 5. Multi-tenancy

- Toutes les requêtes filtrées par `tenant_id`.
- Aucun cache global partagé sans clé tenant.
- Chargement isolé des configurations.

---

### 6. Résilience & robustesse

- Si une règle échoue → log erreur + règle ignorée.
- Ne doit pas bloquer le pipeline.
- Timeout par règle configurable.

---

### 7. Performance

Contraintes :
- Exécution synchrone dans le worker.
- Complexité linéaire par nombre de règles actives.
- Objectif : impact minimal sur SLA < 30s pipeline global.

Optimisations possibles :
- Pré-compilation des règles simples.
- Cache court des configurations tenant.

---

### 8. Sécurité & audit

- Journalisation des règles exécutées.
- Stockage des paramètres utilisés dans `fraud_signal.details`.
- Compatible suppression RGPD (liée au document).

---

### 9. Dépendances

Dépend :
- Module Extraction structurée.
- Module Configuration Tenant.
- Module Audit & Historisation.

Expose :
- Signaux vers Aggregation Engine.
- Données explicatives vers API et Human-in-the-Loop.