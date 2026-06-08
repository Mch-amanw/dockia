## Fonctionnalité : OCR multi-format

### 1. Objectif métier
Permettre la conversion fiable de documents hétérogènes (PDF natifs, PDF scannés, images JPG/JPEG/PNG) en texte exploitable et structuré, afin d’alimenter le moteur d’extraction documentaire et, en aval, le moteur de scoring de fraude.

Cette fonctionnalité constitue la première étape essentielle du pipeline d’extraction : sans OCR fiable, aucune donnée structurée exploitable ne peut être produite.

---

### 2. Périmètre fonctionnel (MVP)

#### 2.1 Formats supportés
- PDF natifs (texte intégré)
- PDF scannés (images encapsulées)
- Images : JPG, JPEG, PNG

Tout document reçu via l’interface web ou l’API REST et accepté par le système doit être traité par le module OCR.

---

### 3. Comportement attendu

#### 3.1 Détection du type technique du document
- Détection automatique si le PDF contient du texte exploitable.
- Si texte natif présent : extraction directe du texte.
- Si document image ou PDF scanné : déclenchement du moteur OCR.

Cette logique est transparente pour l’utilisateur.

---

#### 3.2 Extraction du contenu textuel
Pour chaque document, l’OCR doit produire :
- Le texte brut complet du document.
- Une structuration minimale (blocs, lignes ou bounding boxes si disponibles).
- Un indicateur de confiance global.

Le texte doit être normalisé (encodage UTF-8, suppression des artefacts évidents).

---

#### 3.3 Indicateur de qualité
Chaque traitement OCR doit fournir :
- Un score de confiance global.
- La possibilité d’identifier des zones à faible confiance (si supporté par le moteur).

Ces informations sont utilisées par :
- Le module d’extraction structurée.
- L’interface Human-in-the-Loop (affichage des zones incertaines).
- Le moteur de scoring (signal indirect de qualité documentaire).

---

#### 3.4 Gestion des erreurs
Cas gérés :
- Fichier illisible ou corrompu.
- Format non supporté.
- OCR échoué.

Comportement attendu :
- Le job passe en statut d’échec contrôlé.
- L’erreur est journalisée.
- Aucune donnée partielle incohérente n’est propagée.

---

### 4. Règles de gestion
- L’OCR ne modifie jamais le document source.
- Le texte OCR brut est conservé pour audit.
- Chaque résultat OCR est lié à un `tenant_id`.
- La version du moteur OCR utilisée doit être historisée.
- Aucun document n’est partagé entre tenants.

---

### 5. Critères d’acceptation
- Un PDF natif produit un texte cohérent sans passage inutile par OCR image.
- Un PDF scanné produit un texte exploitable.
- Une image JPG/PNG produit un texte exploitable.
- Le texte OCR est stocké et accessible au module d’extraction.
- Un score de confiance est disponible.
- En cas d’erreur, le statut du job reflète correctement l’échec.