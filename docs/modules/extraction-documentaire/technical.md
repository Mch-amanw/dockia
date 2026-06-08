## Module : Extraction Documentaire – Spécification Technique

### 1. Position dans l’architecture
Le module est exécuté dans le pipeline asynchrone côté worker.

Flux technique :
1. Récupération du document depuis le stockage S3.
2. Détection du type et pré-traitement.
3. OCR.
4. Extraction structurée.
5. Normalisation et validation.
6. Persistance des résultats en base PostgreSQL.

Il est implémenté en Python et intégré au service de traitement.

---

### 2. Composants internes

#### 2.1 Pré-traitement
- Conversion PDF → images si nécessaire.
- Amélioration d’image (rotation, redimensionnement, contraste) si requis.

#### 2.2 OCR Engine
- Intégration avec PaddleOCR, Tesseract ou équivalent.
- Extraction du texte brut.
- Extraction des bounding boxes si disponibles.

Résultats stockés :
- Texte complet.
- Structure OCR (JSON).
- Score de confiance global.

---

#### 2.3 Moteur d’extraction
Approche hybride possible :
- Règles (regex, patterns structurés).
- Modèles IA pré-entraînés.

Séparation logique par type de document (ex : extract_invoice(), extract_id(), extract_bank_statement()).

Sortie standardisée :
```
{
  "documentType": "invoice",
  "fields": {
    "invoiceNumber": {"value": "...", "confidence": 0.92},
    ...
  },
  "metadata": {
    "ocrConfidence": 0.88,
    "modelVersion": "vX.Y"
  }
}
```

---

### 3. Modélisation des données
En base PostgreSQL :
- Table `extractions`
  - id
  - job_id
  - tenant_id
  - document_type
  - raw_ocr_text
  - structured_data (JSONB)
  - ocr_confidence
  - extraction_model_version
  - created_at

Index sur :
- tenant_id
- job_id

---

### 4. Multi-tenancy
- Toutes les requêtes incluent un filtre par `tenant_id`.
- Aucune mutualisation de données entre tenants.

---

### 5. Intégrations
- Stockage S3 : récupération du document source.
- Base PostgreSQL : persistance des résultats.
- Module scoring fraude : transmission des données structurées.
- Module audit : journalisation des versions et métadonnées.

---

### 6. Contraintes non fonctionnelles
- Temps cible d’extraction compatible avec objectif < 30 secondes par document.
- Scalabilité horizontale via workers.
- Tolérance aux erreurs OCR (gestion d’exception, statut d’échec du job).
- Journalisation détaillée pour audit.

---

### 7. Versioning et audit
- Stockage explicite de la version OCR.
- Stockage explicite de la version du modèle d’extraction.
- Conservation du texte OCR brut pour auditabilité.

---

### 8. Sécurité
- Traitement en environnement sécurisé.
- Données chiffrées au repos (via stockage et base).
- Transmission interne sécurisée (TLS si services distribués).
- Respect des politiques de rétention configurées par tenant.