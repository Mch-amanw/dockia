## Fonctionnalité : Moteur de règles configurables

### 1. Objectif métier
Permettre à chaque tenant de définir, activer et paramétrer ses propres **règles métier de détection de fraude**, afin d’adapter le scoring aux spécificités réglementaires, opérationnelles et sectorielles.

Le moteur de règles contribue aux sous-scores et au score global du module **Moteur de Détection & Scoring Fraude**, tout en garantissant explicabilité et auditabilité.

---

### 2. Périmètre
La fonctionnalité couvre :
- L’exécution de règles métier déterministes.
- La configuration et la pondération des règles par tenant.
- L’activation/désactivation granulaire.
- La production de signaux explicables.
- La traçabilité des règles appliquées et de leur version.

Hors périmètre :
- L’entraînement ou la gestion des modèles IA.
- Les décisions automatiques (elles dépendent du score global agrégé).

---

### 3. Typologie des règles (MVP)
Les règles peuvent porter sur :

1. **Cohérence des données extraites**  
   - Ex : total facture ≠ somme des lignes.
   - Ex : date d’émission postérieure à la date actuelle.

2. **Correspondance inter-champs**  
   - Ex : nom incohérent entre plusieurs zones du document.
   - Ex : IBAN invalide selon format attendu.

3. **Contrôles structurels**  
   - Ex : absence d’un champ obligatoire.
   - Ex : format invalide (numéro d’identification, date, etc.).

4. **Règles contextuelles tenant**  
   - Ex : seuil de montant maximum.
   - Ex : pays interdits.

Chaque règle :
- Est associée à un ou plusieurs types de documents.
- Contribue à un sous-score spécifique.
- Possède un poids configurable.

---

### 4. Configuration par tenant
Chaque tenant peut :
- Activer/désactiver une règle.
- Modifier les paramètres (seuils, listes, valeurs attendues).
- Ajuster la pondération dans la catégorie concernée.
- Définir un niveau de sévérité.

Contraintes :
- Aucune configuration n’impacte un autre tenant.
- Les modifications ne sont pas rétroactives.
- Toute modification est historisée et versionnée.

---

### 5. Exécution des règles
Lors du traitement d’un document :
1. Chargement de la configuration du tenant.
2. Sélection des règles applicables selon le type de document.
3. Exécution déterministe.
4. Production d’un ou plusieurs signaux si déclenchement.
5. Attribution d’une contribution au sous-score concerné.

Une règle peut :
- Ne pas se déclencher.
- Générer un signal faible.
- Générer un signal fort selon la gravité.

---

### 6. Sorties fonctionnelles
Pour chaque règle déclenchée :
- Code unique.
- Description explicite.
- Catégorie de sous-score.
- Sévérité.
- Contribution au score.
- Paramètres utilisés.
- Version de la règle.

Ces éléments doivent être exploitables par :
- L’API REST.
- L’interface Human-in-the-Loop.
- Le module d’audit.

---

### 7. Règles de gestion
- Les règles sont purement déterministes.
- Une règle doit être reproductible à partir des données historisées.
- L’absence de données nécessaires ne doit pas provoquer d’erreur bloquante (fallback contrôlé).
- Les règles ne produisent jamais un verdict binaire définitif.
- Le moteur doit fonctionner même si aucune règle n’est active.

---

### 8. Critères d’acceptation
- ✅ Un tenant peut activer/désactiver une règle sans impacter les autres.
- ✅ Une règle paramétrée avec un seuil personnalisé est appliquée correctement.
- ✅ Les règles déclenchées apparaissent dans les signaux explicatifs.
- ✅ La version des règles est enregistrée dans l’analyse.
- ✅ La reproduction d’un scoring passé avec la même version produit le même résultat.