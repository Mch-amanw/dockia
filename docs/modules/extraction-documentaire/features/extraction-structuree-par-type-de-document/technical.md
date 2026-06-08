## Spécification technique : Extraction structurée par type de document

### 1. Position dans le flux technique
Cette fonctionnalité est exécutée dans le worker asynchrone, après :
1. Récupération du document depuis S3.
2. Exécution de l’OCR.

Elle consomme :
- `raw_ocr_text`
- `ocr_structure` (si disponible)
- `document_type`

Elle produit :
- `structured_data` (JSONB)
- Indicateurs de validation
- Métadonnées de version

---

### 2. Architecture interne

#### 2.1 Séparation par type
Implémentation logique sous forme de fonctions ou classes dédiées :
- `extract_invoice(data)`
- `extract_id(data)`
- `extract_bank_statement(data)`

Chaque extracteur :
- Applique règles (regex, patterns).
- Peut appeler un modèle IA pré-entraîné.
- Retourne une structure normalisée commune.

---

#### 2.2 Format de sortie standardisé
Structure cible (exemple) :

```json
{
  "documentType": "invoice",
  "fields": {
    "invoiceNumber": {
      "value": "F-2024-001",
      "confidence": 0.92,
      "position": {"page": 1}
    }
  },
  "validations": {
    "totalConsistency": true
  },
  "metadata": {
    "ocrConfidence": 0.88,
    "extractionModelVersion": "v1.0"
  }
}
```

Contraintes :
- Format homogène pour tous les types.
- Champs absents explicitement omis ou marqués `null`.

---

### 3. Normalisation technique

#### 3.1 Dates
- Parsing multi-format.
- Conversion vers ISO 8601.
- Gestion des erreurs avec fallback contrôlé.

#### 3.2 Montants
- Suppression séparateurs locaux.
- Conversion en type numérique (Decimal recommandé).
- Stockage en JSON numérique.

#### 3.3 IBAN
- Nettoyage espaces.
- Validation structurelle basique.

---

### 4. Persistance
Résultats enregistrés dans la table `extractions` :
- `tenant_id`
- `job_id`
- `document_type`
- `raw_ocr_text`
- `structured_data` (JSONB)
- `ocr_confidence`
- `extraction_model_version`
- `created_at`

Index obligatoires sur `tenant_id` et `job_id`.

---

### 5. Intégrations
- Module OCR : source des données.
- Module scoring fraude : consommation de `structured_data` et des validations.
- Module audit : enregistrement des versions modèles et règles.
- API REST : exposition des résultats via endpoint de récupération de job.

---

### 6. Contraintes techniques
- Compatible avec traitement < 30 secondes global.
- Scalabilité horizontale via workers.
- Gestion robuste des exceptions (échec extraction → statut job mis à jour).
- Journalisation détaillée (logs structurés).

---

### 7. Versioning
- Chaque exécution inclut `extractionModelVersion`.
- Version OCR héritée des métadonnées amont.
- Aucune suppression du texte OCR brut (auditabilité complète).

---

### 8. Sécurité et multi-tenant
- Toutes les opérations incluent `tenant_id`.
- Aucune donnée en mémoire ou en base partagée entre tenants.
- Respect des politiques de rétention configurées.
- Données persistées dans PostgreSQL avec chiffrement au repos (niveau infrastructure).