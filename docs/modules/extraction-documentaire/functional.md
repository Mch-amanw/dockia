## Module : Extraction Documentaire

### 1. Rôle du module dans Dockia
Le module d’Extraction Documentaire est responsable de la transformation d’un document brut (PDF ou image) en données structurées exploitables par le moteur de scoring et par l’interface Human-in-the-Loop.

Il intervient après l’upload et la création du job, et fournit :
- Le texte issu de l’OCR.
- Les champs structurés extraits selon le type de document.
- Un niveau de confiance par champ.
- Les métadonnées nécessaires à l’audit et à l’explicabilité.

Ce module alimente directement le moteur de détection de fraude et l’interface analyste.

---

### 2. Périmètre fonctionnel (MVP)

#### 2.1 Types de documents pris en charge
- Factures
- Pièces d’identité (CNI, passeports, permis)
- Relevés bancaires

L’extraction doit être adaptée dynamiquement au type de document identifié ou fourni.

---

### 3. Fonctionnalités principales

#### 3.1 OCR multi-format
- Support des PDF natifs.
- Support des PDF scannés.
- Support des images JPG, JPEG, PNG.
- Extraction du texte brut.
- Extraction de la structure spatiale (blocs, lignes, coordonnées si disponibles).

Sorties :
- Texte complet normalisé.
- Données OCR structurées (si disponibles).
- Indicateur de qualité OCR (ex: confiance globale).

---

#### 3.2 Classification ou validation du type de document
- Utilisation du type fourni par l’API ou détermination automatique si non fourni.
- Validation de cohérence entre le type déclaré et le contenu détecté.

En cas d’incohérence, un indicateur est transmis au module de scoring.

---

#### 3.3 Extraction structurée par type

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
- Valeur extraite
- Score de confiance
- Position dans le document (si disponible)

---

#### 3.4 Normalisation et validation métier
- Normalisation des dates (format ISO).
- Normalisation des montants (format numérique standardisé).
- Normalisation des devises.
- Nettoyage des caractères spéciaux.

Contrôles simples :
- Format valide (date, IBAN, numéro).
- Cohérence arithmétique simple (ex : total HT + TVA ≈ TTC).

Les incohérences détectées sont transmises au moteur de fraude.

---

#### 3.5 Gestion des incertitudes
- Chaque champ inclut un score de confiance.
- Les champs manquants ou ambigus sont explicitement signalés.
- Aucune donnée n’est inventée si absente.

---

#### 3.6 Interaction avec Human-in-the-Loop
- Mise à disposition des champs extraits dans l’interface analyste.
- Affichage des niveaux de confiance.
- Support de la correction manuelle.
- Conservation de la version initiale extraite pour audit.

---

### 4. Règles de gestion
- L’extraction ne produit jamais de verdict de fraude.
- Les données extraites doivent être traçables jusqu’au document source.
- Chaque extraction est liée à un `tenant_id`.
- Les versions de modèles utilisées doivent être historisées.
- Les données d’un tenant ne doivent jamais être exposées à un autre.