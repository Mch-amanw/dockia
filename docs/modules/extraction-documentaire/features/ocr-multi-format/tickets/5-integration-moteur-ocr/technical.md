## Ticket : Intégration moteur OCR – Spécification Technique

### 1. Positionnement dans l’architecture
L’intégration du moteur OCR s’effectue dans le worker du pipeline asynchrone du module Extraction Documentaire.

Flux concerné :
1. Récupération du document depuis S3.
2. Conversion en image(s) si nécessaire (hors implémentation principale de ce ticket si déjà existante).
3. Appel au moteur OCR.
4. Transformation de la sortie brute du moteur en structure standardisée interne.
5. Retour de la structure au module d’extraction.

---

### 2. Intégration du moteur OCR

#### 2.1 Choix et encapsulation
- Intégration de PaddleOCR ou équivalent.
- Encapsulation dans un composant dédié (ex: `OcrService`).
- Isolation de la dépendance externe pour permettre un changement futur de moteur.

Interface interne recommandée :

```python
class OcrService:
    def extract(self, images: List[Image]) -> OcrResult:
        ...
```

---

#### 2.2 Gestion multi-pages
- Pour les PDF convertis en images : traitement page par page.
- Agrégation des résultats dans une structure unique.
- Conservation de l’ordre des pages.

---

### 3. Format de sortie standardisé
Le résultat OCR doit être transformé vers un format interne unifié :

```json
{
  "rawText": "...",
  "pages": [
    {
      "pageNumber": 1,
      "text": "...",
      "blocks": [
        {
          "text": "...",
          "confidence": 0.93,
          "boundingBox": [x1, y1, x2, y2]
        }
      ]
    }
  ],
  "confidence": 0.87,
  "ocrEngineVersion": "x.y.z"
}
```

Contraintes :
- `rawText` = concaténation normalisée du texte de toutes les pages.
- `confidence` = score global (moyenne ou agrégation cohérente).
- `ocrEngineVersion` récupérée dynamiquement si possible ou définie via configuration.

---

### 4. Normalisation
- Encodage UTF-8 obligatoire.
- Suppression des caractères non imprimables.
- Normalisation des retours à la ligne.

---

### 5. Persistance
Les données issues de l’OCR doivent être transmises au module responsable de la persistance dans la table `extractions` :
- `raw_ocr_text`
- `structured_data` (incluant structure OCR si utilisée)
- `ocr_confidence`
- `ocr_engine_version`
- `tenant_id`
- `job_id`

Aucune écriture partielle ne doit être effectuée en cas d’échec OCR.

---

### 6. Gestion des erreurs et robustesse
- Encapsulation des exceptions du moteur OCR.
- Logging structuré (incluant `tenant_id`, `job_id`).
- Remontée d’erreur contrôlée au pipeline.
- Nettoyage mémoire des images après traitement.

---

### 7. Contraintes non fonctionnelles
- Compatible avec objectif global < 30 secondes par document.
- Scalabilité horizontale via workers Docker.
- Consommation mémoire maîtrisée pour documents multi-pages.
- Aucune persistance locale permanente des fichiers.

---

### 8. Sécurité et multi-tenancy
- Le `tenant_id` est propagé dans le contexte d’exécution.
- Aucune mutualisation de cache ou de données OCR entre tenants.
- Respect des politiques de chiffrement existantes (S3 et PostgreSQL).

---

### 9. Dépendances
- Stockage S3 (lecture document).
- Bibliothèque OCR (PaddleOCR ou équivalent).
- Module Extraction Documentaire (consommateur du résultat).
- Module Audit (historisation version moteur).