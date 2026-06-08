## Ticket : Moteur de règles paramétrables par tenant

### 1. Objectif
Implémenter le cœur du **moteur de règles métier paramétrables par tenant** permettant :

- L’exécution de règles déterministes de détection de fraude.
- La prise en compte de paramètres spécifiques à chaque tenant (seuils, listes, poids, sévérité).
- La production de signaux explicables contribuant au scoring global.

Ce ticket couvre la capacité du système à charger dynamiquement les configurations par tenant et à appliquer les règles correspondantes lors du traitement d’un document.

---

### 2. Périmètre fonctionnel

Inclus :
- Catalogue interne de règles disponibles (registry plateforme).
- Configuration spécifique des règles par tenant.
- Activation / désactivation par règle.
- Paramétrage de seuils et valeurs métier.
- Pondération par règle.
- Production de signaux exploitables par le module d’agrégation.
- Historisation de la version des règles utilisées.

Exclus :
- Interface d’administration avancée (UI complète).
- Gestion des modèles IA.
- Décision automatique finale (gérée par l’Aggregation Engine).

---

### 3. Comportement attendu

#### 3.1 Chargement des règles
Lors du traitement d’un document :
1. Le moteur identifie le `tenant_id`.
2. Il charge la configuration active des règles pour ce tenant.
3. Il filtre les règles applicables selon le type de document.

Si aucune règle n’est active :
- Le moteur retourne un résultat vide sans erreur bloquante.

---

#### 3.2 Paramétrage par tenant
Pour chaque règle, un tenant peut définir :

- `enabled` (activation/désactivation)
- `parameters` (ex : seuil montant maximum, liste pays interdits)
- `weight`
- `severity_level`

Les modifications :
- Ne sont pas rétroactives.
- S’appliquent uniquement aux analyses futures.
- Sont versionnées.

---

#### 3.3 Exécution des règles
Chaque règle :
- Est déterministe.
- S’appuie uniquement sur les données extraites et métadonnées disponibles.
- Retourne un résultat structuré indiquant si elle est déclenchée.

Si déclenchée, elle produit :
- Code unique.
- Description.
- Catégorie de sous-score.
- Contribution normalisée.
- Détails explicatifs.
- Version utilisée.

Une règle ne peut jamais produire un verdict binaire de fraude.

---

#### 3.4 Contribution au scoring

- Chaque règle déclenchée contribue à un sous-score.
- La contribution est transmise à l’Aggregation Engine.
- Les poids configurés sont pris en compte.

Le moteur ne calcule pas le score global final mais fournit les éléments nécessaires.

---

#### 3.5 Explicabilité & audit
Pour chaque règle exécutée, le système doit permettre de retrouver :

- Les paramètres utilisés.
- La version de la règle.
- Le résultat produit.
- Le contexte du document.

Ces informations sont persistées pour audit et affichage Human-in-the-Loop.

---

### 4. Contraintes fonctionnelles

- Isolation stricte par `tenant_id`.
- Exécution robuste (une règle en erreur ne bloque pas le traitement global).
- Reproductibilité complète avec même version + même configuration.
- Fonctionnement même si certaines données sont absentes.
- Aucune dépendance aux modèles IA.

---

### 5. Dépendances

Dépend :
- Module Extraction structurée.
- Module Configuration Tenant.
- Module Audit & Historisation.

Alimente :
- Aggregation Engine.
- API REST.
- Interface Human-in-the-Loop.