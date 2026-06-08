# Spécification Technique

## 1. Position dans l’architecture

Le module est intégré dans le pipeline asynchrone :

1. OCR
2. Extraction
3. ➜ Moteur de Détection & Scoring
4. Stockage résultats

Il est exécuté dans le worker de traitement ou via un service dédié interne.

---

## 2. Architecture interne

### 2.1 Composants principaux

1. **Rule Engine**
   - Exécution des règles métier
   - Système modulaire par type de document
   - Retourne liste des règles déclenchées + scores partiels

2. **AI Scoring Engine**
   - Appel aux modèles IA (anomalies, génération IA, etc.)
   - Retour probabiliste
   - Versioning explicite des modèles

3. **Aggregation Engine**
   - Normalisation des scores
   - Pondération configurable
   - Calcul des sous-scores
   - Calcul du score global

4. **Explainability Builder**
   - Consolidation des facteurs
   - Attribution de contribution
   - Formatage pour stockage et API

---

## 3. Modélisation des données (conceptuelle)

### Entités principales

- `fraud_analysis`
  - id
  - job_id
  - tenant_id
  - global_score
  - confidence_level
  - model_version
  - rules_version
  - created_at

- `fraud_subscore`
  - analysis_id
  - category
  - score
  - weight

- `fraud_signal`
  - analysis_id
  - type (rule / ai)
  - code
  - description
  - contribution

Toutes les entités incluent `tenant_id`.

---

## 4. Multi-tenancy

- Filtrage strict par `tenant_id`.
- Chargement dynamique des configurations.
- Isolation logique garantie au niveau base de données.

---

## 5. Versioning

Le module doit enregistrer :

- Version des modèles IA utilisés.
- Version des règles.
- Version du moteur (si évolution majeure).

Les résultats doivent être reproductibles à partir des versions historisées.

---

## 6. Performance & Scalabilité

Contraintes :

- Intégration dans pipeline < 30 secondes globalement.
- Support de montée en charge horizontale.
- Traitement indépendant par document.
- Stateless (configuration chargée dynamiquement).

---

## 7. Sécurité & conformité

- Aucune donnée inter-tenant exposée.
- Journalisation complète des exécutions.
- Conservation des éléments nécessaires à l’audit RGPD.
- Possibilité de suppression conforme aux politiques de rétention.

---

## 8. Dépendances

Dépend :
- Module OCR
- Module Extraction structurée
- Module Configuration Tenant
- Module Audit & Historisation

Expose ses résultats à :
- API REST
- Module Human-in-the-Loop
- Tableau de bord

---

## 9. Contraintes

- Doit rester explicable (pas de modèle opaque sans facteur explicatif).
- Doit fonctionner même si certaines catégories de signaux sont indisponibles.
- Doit permettre l’ajout futur de nouvelles catégories de sous-scores.

---