## Ticket : Mapping des champs par type documentaire

### 1. Objectif
Définir et implémenter les schémas de mapping des champs extraits pour chaque type documentaire pris en charge dans le MVP :
- Factures
- Pièces d’identité (CNI, passeports, permis)
- Relevés bancaires

Ce ticket vise à formaliser un référentiel unique des champs attendus par type de document, incluant :
- Le nom canonique du champ (clé technique exposée via API).
- Son type de donnée.
- Son caractère obligatoire ou optionnel.
- Les règles de normalisation associées.
- Les validations métier simples associées.

Ce mapping constitue la base contractuelle entre :
- Le moteur d’extraction.
- Le moteur de scoring fraude.
- L’API REST.
- L’interface Human-in-the-Loop.

---

### 2. Principes généraux

- Chaque type documentaire possède un schéma de champs dédié.
- Les noms de champs sont standardisés et stables dans le temps (compatibilité API).
- Aucun champ non défini dans le mapping ne doit être exposé dans `structured_data`.
- Les champs non détectés doivent être explicitement présents avec une valeur `null` ou un indicateur d’absence.
- Chaque champ est accompagné d’un score de confiance.
- Les formats exposés sont déjà normalisés (dates ISO, montants numériques, IBAN nettoyé, etc.).

---

### 3. Mapping – Factures

#### 3.1 Champs principaux
- `supplierName` (string)
- `invoiceNumber` (string)
- `issueDate` (date ISO)
- `dueDate` (date ISO, optionnel)
- `amountExclTax` (number)
- `taxAmount` (number, optionnel si non applicable)
- `amountInclTax` (number)
- `currency` (string, format ISO 4217 si identifiable)
- `iban` (string, optionnel)

#### 3.2 Validations associées
- Cohérence arithmétique simple : `amountExclTax + taxAmount ≈ amountInclTax`.
- Format valide pour dates et IBAN.

---

### 4. Mapping – Pièces d’identité

#### 4.1 Champs principaux
- `lastName` (string)
- `firstName` (string)
- `dateOfBirth` (date ISO)
- `placeOfBirth` (string)
- `documentNumber` (string)
- `expirationDate` (date ISO)
- `nationality` (string)

#### 4.2 Validations associées
- Format valide des dates.
- Cohérence basique : date d’expiration postérieure à date d’émission si détectée.

---

### 5. Mapping – Relevés bancaires

#### 5.1 Champs principaux
- `accountHolderName` (string)
- `iban` (string)
- `statementPeriodStart` (date ISO)
- `statementPeriodEnd` (date ISO)
- `openingBalance` (number)
- `closingBalance` (number)
- `transactions` (array)

#### 5.2 Structure d’une transaction
Chaque élément de `transactions` contient :
- `date` (date ISO)
- `label` (string)
- `amount` (number)

#### 5.3 Validations associées
- Format IBAN valide.
- Cohérence simple possible entre soldes et transactions (si calculable).

---

### 6. Structure commune par champ
Pour chaque champ (hors tableaux complexes) :
- `value` : valeur normalisée ou `null`.
- `confidence` : score entre 0 et 1.
- `position` : métadonnée facultative issue de l’OCR (page, coordonnées si disponibles).

---

### 7. Contraintes de cohérence globale

- Le mapping doit être compatible avec le format standard `structured_data` défini dans la fonctionnalité parente.
- Les noms de champs doivent être homogènes et cohérents entre les types.
- Le mapping ne doit pas inclure de logique de scoring ou de détection de fraude.
- Toute évolution future devra être versionnée (compatibilité ascendante).