## Fonctionnalité : Extraction structurée par type de document

### 1. Objectif métier
Permettre la transformation d’un document brut (après OCR) en données structurées, fiables et normalisées, adaptées à son type (facture, pièce d’identité, relevé bancaire), afin d’alimenter :
- Le moteur de détection et scoring de fraude.
- L’interface Human-in-the-Loop.
- Les intégrations API clients.

L’objectif est de fournir des champs exploitables, accompagnés d’un niveau de confiance et traçables jusqu’au document source.

---

### 2. Périmètre

Types de documents pris en charge (MVP) :
- Factures
- Pièces d’identité (CNI, passeports, permis)
- Relevés bancaires

Entrée :
- Texte OCR brut
- Structure OCR (blocs, lignes, coordonnées si disponibles)
- Type de document (fourni ou déterminé en amont)

Sortie :
- Données structurées par type
- Score de confiance par champ
- Métadonnées associées (version modèle, confiance OCR)

---

### 3. Règles de gestion

#### 3.1 Adaptation au type de document
L’extraction doit :
- Appliquer un schéma de champs spécifique au type de document.
- Ne produire que les champs pertinents pour ce type.
- Signaler explicitement les champs attendus mais non détectés.

En cas d’incohérence forte entre le contenu détecté et le type déclaré, un indicateur est ajouté aux métadonnées pour transmission au moteur de fraude.

---

#### 3.2 Champs extraits (MVP)

##### A. Factures
Exemples de champs :
- Nom fournisseur
- Numéro de facture
- Date d’émission
- Date d’échéance
- Montant HT
- Montant TTC
- TVA
- Devise
- IBAN / coordonnées bancaires (si présentes)

##### B. Pièces d’identité
Exemples de champs :
- Nom
- Prénom
- Date de naissance
- Lieu de naissance
- Numéro de document
- Date d’expiration
- Nationalité

##### C. Relevés bancaires
Exemples de champs :
- Nom du titulaire
- IBAN
- Période du relevé
- Solde initial
- Solde final
- Liste structurée des transactions (date, libellé, montant)

Pour chaque champ :
- `value` : valeur extraite
- `confidence` : score de confiance (0–1)
- `source` ou position (si disponible via OCR)

---

#### 3.3 Normalisation des données
Les données extraites doivent être normalisées :
- Dates au format ISO 8601 (YYYY-MM-DD si applicable).
- Montants au format numérique standardisé (séparateur décimal unifié).
- Devise au format standard (ex : EUR, USD si identifiable).
- Nettoyage des caractères parasites issus de l’OCR.

La valeur normalisée est celle exposée via API et utilisée pour le scoring.

---

#### 3.4 Validation métier simple
Des contrôles de cohérence basiques doivent être réalisés :
- Vérification du format des dates.
- Vérification de structure IBAN (si présent).
- Cohérence arithmétique simple (ex : HT + TVA ≈ TTC pour facture).

Les incohérences détectées sont signalées comme indicateurs et transmises au module de détection de fraude, sans bloquer l’extraction.

---

#### 3.5 Gestion des incertitudes
- Aucun champ ne doit être inventé.
- Les champs absents sont explicitement marqués comme manquants.
- Les champs ambigus doivent refléter un niveau de confiance faible.
- L’extraction ne rend jamais de verdict de fraude.

---

### 4. Critères d’acceptation
- Pour un document valide, les champs principaux attendus sont présents dans la structure de sortie.
- Chaque champ comporte une valeur et un score de confiance.
- Les formats normalisés sont conformes (dates ISO, montants numériques).
- Les incohérences simples (ex : total incorrect) sont détectées et signalées.
- Les données sont exploitables par le moteur de scoring sans transformation supplémentaire.
- L’extraction est historisée et liée au `tenant_id` et au `job_id`.

---

### 5. Dépendances
- OCR multi-format (production du texte et des métadonnées spatiales).
- Validation du type de document.
- Module de scoring fraude (consommateur des données structurées).
- Module Human-in-the-Loop (affichage et correction).