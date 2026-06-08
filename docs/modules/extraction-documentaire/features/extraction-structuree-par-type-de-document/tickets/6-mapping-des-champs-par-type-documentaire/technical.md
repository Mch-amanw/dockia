## Spécification technique – Mapping des champs par type documentaire

### 1. Objectif technique
Implémenter des schémas structurés et centralisés décrivant les champs attendus pour chaque type documentaire, utilisés par :
- Les extracteurs (`extract_invoice`, `extract_id`, `extract_bank_statement`).
- La validation métier.
- La génération du JSON `structured_data`.

Ces schémas doivent être définis de manière déclarative et réutilisable.

---

### 2. Définition des schémas

#### 2.1 Structure recommandée
Créer un module dédié, par exemple :

```
extraction/
  schemas/
    invoice_schema.py
    id_schema.py
    bank_statement_schema.py
    base_schema.py
```

Chaque schéma définit :
- Nom du champ (clé API).
- Type attendu (`string`, `date`, `number`, `array`).
- Indicateur `required` (booléen).
- Règles de normalisation à appliquer.
- Validations associées.

Exemple conceptuel (Python) :

```python
FieldDefinition(
    name="invoiceNumber",
    field_type="string",
    required=True,
    normalizer=normalize_string,
    validators=[validate_non_empty]
)
```

---

### 3. Intégration avec les extracteurs

Chaque fonction d’extraction :
- Utilise le schéma correspondant.
- Tente d’extraire les valeurs via règles/IA.
- Passe la valeur brute au normalizer défini dans le schéma.
- Applique les validations simples.
- Génère une entrée standardisée :

```json
"invoiceNumber": {
  "value": "F-2024-001",
  "confidence": 0.92,
  "position": {"page": 1}
}
```

Pour les champs absents :

```json
"invoiceNumber": {
  "value": null,
  "confidence": 0.0
}
```

---

### 4. Gestion des tableaux (transactions)

Le schéma `bank_statement_schema` doit inclure une définition spécifique pour :

- Champ `transactions` de type `array`.
- Sous-schéma transaction : `date`, `label`, `amount`.

Chaque transaction doit suivre la même structure `value + confidence` au niveau de chaque sous-champ.

---

### 5. Validation et normalisation

Les fonctions de normalisation doivent être centralisées (ex : module `normalizers.py`) :
- `normalize_date()` → ISO 8601.
- `normalize_amount()` → Decimal.
- `normalize_iban()` → suppression espaces + uppercase.

Les validations simples (ex : cohérence HT/TVA/TTC) doivent être déclenchées après instanciation complète du schéma et produire un bloc `validations` dans `structured_data`.

---

### 6. Compatibilité avec la persistance

Le résultat final doit respecter le format cible stocké en base dans `extractions.structured_data` (JSONB) :

- `documentType`
- `fields`
- `validations`
- `metadata`

Les clés doivent être strictement conformes au mapping défini.

---

### 7. Versioning

- Associer une constante `EXTRACTION_SCHEMA_VERSION`.
- Inclure cette version dans `metadata.extractionModelVersion`.
- Toute modification du mapping (ajout ou renommage de champ) nécessite incrément de version.

---

### 8. Contraintes

- Aucun accès direct à la base depuis les schémas (logique pure).
- Aucun couplage avec le moteur de scoring.
- Respect strict du `tenant_id` au niveau de l’extraction globale (hérité du pipeline).
- Code testable unitairement (schémas et normalisateurs isolables).

Ce ticket ne couvre pas l’implémentation des modèles IA eux-mêmes, uniquement la définition et l’implémentation des schémas de mapping et de leur intégration dans le moteur d’extraction.