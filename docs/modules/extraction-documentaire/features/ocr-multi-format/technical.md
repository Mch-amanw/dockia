## Spécification Technique – OCR multi-format

### 1. Position dans le pipeline
La fonctionnalité OCR multi-format est exécutée dans le worker du pipeline asynchrone, après récupération du document depuis le stockage S3.

Flux technique :
1. Récupération du fichier depuis S3.
2. Détection du type (PDF natif vs image).
3. Si PDF natif avec texte → extraction directe.
4. Sinon → conversion en image(s) si nécessaire.
5. Passage au moteur OCR.
6. Normalisation et structuration du résultat.
7. Persistance en base PostgreSQL.

---

### 2. Composants techniques

#### 2.1 Détection PDF natif
- Utilisation d’une bibliothèque Python adaptée pour détecter la présence de texte dans un PDF.
- Si texte présent et exploitable : extraction directe sans OCR image.

#### 2.2 Conversion PDF → images
- Pour PDF scannés : conversion page par page en images.
- Gestion multi-pages obligatoire.

#### 2.3 Intégration moteur OCR
- Intégration avec PaddleOCR, Tesseract ou équivalent.
- Paramétrage compatible documents administratifs.
- Support du retour des bounding boxes si disponible.

Sortie attendue (structure interne standardisée) :
```
{
  "rawText": "...",
  "pages": [
    {
      "pageNumber": 1,
      "text": "...",
      "blocks": [...]
    }
  ],
  "confidence": 0.87,
  "ocrEngineVersion": "x.y.z"
}
```

---

### 3. Persistance des données
Stockage en PostgreSQL (table `extractions`) :
- `raw_ocr_text`
- `structured_data` (incluant structure OCR si nécessaire)
- `ocr_confidence`
- `extraction_model_version`
- Version moteur OCR
- `tenant_id`
- `job_id`

Index sur `tenant_id` et `job_id`.

---

### 4. Gestion des erreurs
- Exceptions capturées et journalisées.
- Mise à jour du statut du job en cas d’échec.
- Aucun enregistrement partiel incohérent en base.

---

### 5. Contraintes non fonctionnelles
- Temps de traitement compatible avec objectif global < 30 secondes par document.
- Scalabilité horizontale via workers.
- Consommation mémoire maîtrisée pour documents multi-pages.
- Journalisation détaillée pour audit.

---

### 6. Dépendances et intégrations
Dépendances :
- Stockage S3 (lecture document).
- Base PostgreSQL (persistance résultats).
- Module d’extraction structurée (consommateur du texte OCR).
- Module audit (journalisation version moteur et métadonnées).

---

### 7. Sécurité
- Traitement en environnement isolé par worker.
- Aucune persistance locale non maîtrisée des documents.
- Respect du `tenant_id` à toutes les étapes.
- Données chiffrées au repos via infrastructure existante.