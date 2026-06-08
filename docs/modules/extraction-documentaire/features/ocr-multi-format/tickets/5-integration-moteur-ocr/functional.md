## Ticket : Intégration moteur OCR

### 1. Objectif
Intégrer un moteur OCR (PaddleOCR ou équivalent) dans le module d’Extraction Documentaire afin de permettre l’extraction fiable du texte à partir :
- des PDF scannés,
- des images (JPG, JPEG, PNG),
- et, si nécessaire, des PDF natifs ne contenant pas de texte exploitable.

Ce ticket couvre l’intégration effective du moteur OCR, sa configuration, la production d’un format de sortie standardisé et l’exposition des informations nécessaires aux modules en aval (extraction structurée, scoring, audit, Human-in-the-Loop).

---

### 2. Périmètre fonctionnel

#### 2.1 Moteur OCR
Le système doit :
- Intégrer un moteur OCR compatible documents administratifs.
- Permettre l’extraction du texte brut.
- Permettre la récupération des informations de structure (pages, blocs, lignes, bounding boxes) si disponibles.
- Fournir un indicateur de confiance global.

Le moteur utilisé doit être versionné et identifiable.

---

#### 2.2 Types de documents concernés
Le moteur OCR doit être utilisé pour :
- Les PDF scannés.
- Les images JPG, JPEG, PNG.
- Les PDF natifs sans texte exploitable.

La détection PDF natif vs scanné est hors périmètre de ce ticket si déjà couverte, mais le moteur doit être prêt à traiter des images page par page.

---

#### 2.3 Sortie fonctionnelle attendue
Pour chaque document traité, le moteur OCR doit produire une structure normalisée contenant :
- Le texte brut complet du document.
- Une structuration par page.
- Les blocs ou segments détectés (si disponibles).
- Un score de confiance global.
- La version du moteur OCR utilisée.

Ces données doivent être exploitables par :
- Le moteur d’extraction structurée.
- Le module d’audit.
- L’interface Human-in-the-Loop (pour visualisation des zones incertaines).

---

#### 2.4 Gestion des erreurs
En cas de :
- Fichier illisible,
- Échec du moteur OCR,
- Format non supporté,

Le système doit :
- Lever une erreur contrôlée.
- Ne pas produire de données OCR partielles incohérentes.
- Permettre au pipeline de positionner correctement le statut du job.

---

### 3. Règles de gestion
- Le texte OCR brut doit être conservé pour audit.
- Le moteur OCR ne doit jamais modifier le document source.
- Chaque résultat OCR est associé à un `tenant_id` et à un `job_id`.
- La version du moteur OCR doit être historisée.
- Aucune donnée d’un tenant ne doit être accessible à un autre.