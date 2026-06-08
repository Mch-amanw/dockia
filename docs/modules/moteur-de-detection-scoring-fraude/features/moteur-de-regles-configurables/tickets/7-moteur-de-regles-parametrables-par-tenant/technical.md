## Spécification Technique — Moteur de règles paramétrables par tenant

### 1. Architecture du composant

Implémentation dans le **Rule Engine** du module Moteur de Détection & Scoring.

Sous-composants :
- RuleRegistry
- TenantRuleConfigLoader
- RuleExecutor
- RuleResultBuilder

Flux :
1. Chargement configuration tenant.
2. Sélection règles applicables.
3. Exécution séquentielle.
4. Production liste `RuleResult`.
5. Transmission à Aggregation Engine.

---

### 2. Modélisation des données

#### 2.1 Catalogue des règles (registry applicatif)

Défini côté code :

```python
class BaseRule:
    code: str
    category: str
    version: str

    def evaluate(self, document_data: dict, config: dict) -> RuleResult:
        raise NotImplementedError
```

Chaque règle concrète :
- Implémente `evaluate()`.
- Est enregistrée dans un registry global.

---

#### 2.2 Configuration tenant (PostgreSQL)

Table : `tenant_rule_config`

Champs :
- id (UUID)
- tenant_id (UUID, indexé)
- rule_code (string, indexé)
- enabled (boolean)
- parameters (JSONB)
- weight (float)
- severity_level (string ou int)
- rules_version (string)
- updated_at (timestamp)

Contraintes :
- Unicité (tenant_id, rule_code).
- Index sur tenant_id.

---

### 3. Structure RuleResult

Objet interne :

```python
class RuleResult:
    rule_code: str
    version: str
    triggered: bool
    raw_score: float
    normalized_score: float
    weight: float
    contribution: float
    category: str
    details: dict
```

Règles :
- `normalized_score` borné 0–100.
- `contribution` calculée à partir de normalized_score × weight.

---

### 4. Chargement dynamique

Composant : `TenantRuleConfigLoader`

Responsabilités :
- Charger toutes les règles actives pour un tenant.
- Filtrer par type de document.
- Retourner une configuration prête à exécution.

Option d’optimisation :
- Cache court indexé par (tenant_id, rules_version).
- Invalidation lors mise à jour.

---

### 5. Exécution & résilience

- Exécution synchrone dans le worker.
- Boucle sur règles actives.
- Gestion try/except par règle.
- Log structuré en cas d’erreur.
- Timeout configurable si nécessaire.

Une règle en échec :
- Ne bloque pas le pipeline.
- N’est pas incluse dans les contributions.

---

### 6. Intégration base fraud_analysis

Lors de la persistance :

- Insertion `fraud_signal` pour chaque règle déclenchée.
- Insertion `fraud_subscore` alimentée via Aggregation Engine.
- Stockage de `rules_version` dans `fraud_analysis`.

Le snapshot de configuration doit être identifiable pour audit.

---

### 7. Multi-tenancy

- Toutes les requêtes filtrées par `tenant_id`.
- Aucune donnée de configuration partagée entre tenants.
- Pas de variable globale mutable non isolée.

---

### 8. Performance

Contraintes :
- Complexité O(n) avec n = nombre de règles actives.
- Impact minimal sur pipeline global (< 30s total).
- Stateless (configuration chargée dynamiquement).

---

### 9. Sécurité & audit

- Journalisation des règles exécutées.
- Stockage des paramètres utilisés dans `fraud_signal.details`.
- Compatible suppression RGPD (lié au document).

---

### 10. Dépendances techniques

- FastAPI backend.
- PostgreSQL.
- Worker asynchrone.
- Module Aggregation Engine.
- Module Audit & Historisation.