# 📋 Pratiques de Développement d'Odoo SA
*Analyse approfondie des méthodologies et standards techniques*

---

## 🎯 Vue d'ensemble

Cette analyse compile les pratiques officielles de développement d'Odoo SA, basée sur leur documentation officielle, leurs repositories GitHub, et leurs conférences techniques.

---

## 🌿 1. Standards Git et Gestion de Versions

### Structure des Messages de Commit

**Format obligatoire :**
```
[TAG] module: description courte (< 50 caractères)

Description détaillée expliquant le POURQUOI du changement,
incluant la logique métier et les choix techniques.
```

### Tags de Commit Standards

- **`[FIX]`** : Corrections de bugs (versions stables et développement)
- **`[REF]`** : Refactoring massif d'une fonctionnalité
- **`[IMP]`** : Améliorations de fonctionnalités existantes
- **`[ADD]`** : Nouvelles fonctionnalités
- **`[REM]`** : Suppression de code/fonctionnalités
- **`[MERGE]`** : Fusion de branches

### Règles de Commit

1. **Une règle de validation** : Le message doit former une phrase valide avec "if applied, this commit will [message]"
2. **Scope modulaire** : Éviter les modifications cross-modules dans un même commit
3. **Focus sur le POURQUOI** : Expliquer la raison du changement, pas seulement ce qui change
4. **Configuration Git requise** : `user.name` et `user.email` obligatoires

### Workflow de Branches

- **Branches stables** : 16.0, 17.0, 18.0 (LTS)
- **Forward porting** : Les fixes dans les versions LTS sont automatiquement portées vers les versions supérieures
- **Master** : Pour les nouvelles fonctionnalités uniquement
- **Ratio valeur/risque** : Changements minimaux dans les versions stables

---

## 🏗️ 2. Architecture et Conventions de Code

### Structure des Modules

```
module_name/
├── __manifest__.py
├── models/
├── views/
├── controllers/
├── static/
│   ├── src/
│   └── lib/
├── security/
├── data/
├── demo/
└── tests/
```

### Conventions Python

**Principes fondamentaux :**
- **"Propagate the context"** : Transmettre le contexte dans les appels de méthodes
- **"Think extendable"** : Concevoir pour l'extensibilité
- **"Never commit the Transaction"** : Ne jamais commit la transaction en cours
- **PEP8 + adaptations Odoo** : Respect des standards avec spécificités Odoo

### Conventions XML

- **IDs XML significatifs** : Nommage cohérent et descriptif
- **Formatage consistant** : Indentation et structure uniformes
- **Conventions de sécurité** : Standards pour les règles d'accès

### Conventions JavaScript/CSS

- **Organisation des fichiers statiques** : Structure systématique
- **Variables CSS** : Utilisation stratégique des variables
- **Mixins SCSS** : Conventions de nommage spécifiques

---

## 🔄 3. Processus de Développement et Review

### Workflow GitHub

1. **Fork/Clone** du repository
2. **Branche feature** avec nom descriptif
3. **Pull Request** avec description complète
4. **Review automatique** par l'équipe Odoo
5. **Merge** après validation

### Critères de Pull Request

**Requis :**
- ✅ CLA (Contributor License Agreement) signé
- ✅ Branche cible correcte
- ✅ Style de code respecté
- ✅ Changements minimaux et focalisés
- ✅ Tests inclus si possible
- ✅ "Allow edits from maintainer" coché

**Processus de Review :**
- **Assignment automatique** d'un reviewer Odoo
- **Commentaires/Questions** traités obligatoirement
- **Merge post-validation** avec forward-porting automatique

### Bots CI/CD

- **Robodoo** : Gestion des statuts CI et intégration des commits
- **Runbot** : Tests automatisés et environnements de test
- **Nightly builds** : Tests supplémentaires (modules, localisation, installations isolées)

---

## 📝 4. Gestion des Issues et Organisation

### Reporting d'Issues

**Template obligatoire :**
```
**Versions impactées :** 
**Étapes de reproduction :**
1. 
2. 
3. 

**Comportement actuel :**
**Comportement attendu :**
```

**Règles :**
- ✅ Recherche d'issues existantes avant création
- ✅ Vérification sur la dernière version d'Odoo
- ✅ Étapes de reproduction détaillées
- ❌ Pas de PR de traduction sur GitHub (utiliser Transifex)

### Gestion de Projet

**Outils natifs :**
- Suivi des jalons majeurs
- Intégration budgets/timesheets/achats/ventes
- Sous-tâches et dépendances
- Dashboards personnalisables

**Extensions Agile/Scrum :**
- Apps tiers pour méthodologie Scrum
- Product Backlog, Sprints, Releases
- Velocity charts et rétrospectives
- Sprints time-boxés (généralement 1 mois)

---

## 🧪 5. Pratiques de Tests et CI/CD

### Types de Tests

1. **Tests Python** : Logique métier des modèles
   ```python
   # Tests indépendants des données
   # Pas d'impact sur la base de données
   # Jamais de cr.commit() dans un test
   ```

2. **Tests JavaScript** : Code JS isolé avec QUnit
   ```javascript
   // Tests focalisés et isolés
   // Utilisation de fonctions helper
   // Couverture de différents scénarios
   ```

3. **Tours de Test** : Simulation d'interactions utilisateur
   ```python
   # Tests d'intégration Python/JavaScript
   # Simulation de situations réelles
   # Screenshots/screencasts de debug
   ```

### Organisation des Tests

```
module_name/
└── tests/
    ├── __init__.py
    ├── test_models.py
    ├── test_flows.py
    └── test_ui.py
```

### CI/CD avec Odoo.sh

**Fonctionnalités :**
- **Déploiement instantané** de chaque commit
- **Branches de staging** avec données de production
- **Historique détaillé** et logs en temps réel
- **Tests automatisés** sur des milliers de cas
- **Environnements éphémères** pour les tests

**Intégration :**
- Connexion GitHub native
- Branches dev illimitées
- Tests automatiques sur chaque commit
- Pipeline de déploiement automatisé

---

## 📚 6. Standards de Documentation

### Format et Structure

**Format :** reStructuredText (RST)
```rst
Titre Principal
===============

Sous-titre
----------

* Maximum 100 caractères par ligne
* Espaces uniquement (jamais de tabs)
* 3 espaces pour l'indentation
```

### Conventions de Style

**Langue :** Anglais américain
**Casse :** Sentence case pour les titres
**Apps :** Capitalisation des noms d'apps (ex: Odoo Sales)
**Liens :** Relatifs pour les URLs internes

### Guidelines de Contenu

- **Éviter la duplication** : Référencer plutôt que répéter
- **Style consistant** : Correspondre au ton existant
- **Alt tags obligatoires** pour toutes les images
- **Organisation par dossier** : Un dossier média par fichier RST

### Processus de Contribution

**Changements mineurs :** Interface GitHub
**Changements majeurs :** Git + copie locale
**Review :** Assignment automatique d'un membre Odoo
**Publication :** En ligne le jour suivant l'approbation

### Structure de la Documentation

```
documentation/
├── user/          # Documentation utilisateur
├── developer/     # Documentation développeur
│   ├── tutorials/
│   ├── howtos/
│   └── reference/
├── administration/
└── contributing/
    ├── development/
    └── documentation/
```

---

## 🛠️ 7. APIs et Intégration

### Architecture API

**Une seule API :** RPC uniquement
- **XML-RPC** : Interface native
- **JSON-RPC** : Interface alternative
- **Pas de REST API** natif (extensions tiers possibles)

**Endpoint principal :**
```
xmlrpc/2/object
```

**Méthode principale :**
```python
execute_kw(database, uid, password, model, method, args, kwargs)
```

### Standards d'Intégration

- **Authentification** : Via login/password ou API keys
- **Modèles** : Accès via ORM distant
- **CRUD** : Create, Read, Update, Delete via RPC
- **Recherche** : Domain expressions pour les filtres

---

## 🎯 8. Bonnes Pratiques pour Petites Équipes

### Standards Minimaux

1. **Git :**
   - Format de commit obligatoire avec tags
   - Une fonctionnalité = une branche
   - Messages expliquant le POURQUOI

2. **Code :**
   - Structure modulaire respectée
   - Tests unitaires systématiques
   - Code review avant merge

3. **Documentation :**
   - README par module
   - Docstrings sur les méthodes importantes
   - Commentaires pour les parties complexes

4. **Tests :**
   - Au moins un test par fonctionnalité
   - Tests d'intégration pour les workflows
   - Pas de cr.commit() dans les tests

### Outils Recommandés

- **Odoo.sh** pour l'hébergement et CI/CD
- **GitHub** pour la gestion de code
- **Transifex** pour les traductions
- **Runbot** pour les tests automatisés

### Processus Simplifié

```
Feature Request → Branch → Code + Tests → PR → Review → Merge → Deploy
```

---

## 📈 9. Méthodologie de Développement

### Principes Agile Adaptés

- **Sprints de 1 mois** maximum
- **User Stories** priorisées
- **Releases fréquentes** (1-3 sprints)
- **Rétrospectives** après chaque sprint

### Workflow Interne

1. **Planning** : Définition des User Stories
2. **Development** : Branches feature par story
3. **Review** : Code review obligatoire
4. **Testing** : Tests automatisés + manuels
5. **Release** : Déploiement automatisé
6. **Retrospective** : Amélioration continue

---

## 🔐 10. Sécurité et Qualité

### Standards de Sécurité

- **Vulnerability disclosure** : Processus séparé pour les failles
- **Code review** : Obligatoire avant merge
- **Tests de sécurité** : Intégrés au CI/CD
- **Permissions** : Règles d'accès granulaires

### Qualité du Code

- **Linting automatique** : Respect des conventions
- **Tests de couverture** : Mesure de la couverture
- **Performance monitoring** : Suivi des requêtes
- **Debt technique** : Refactoring régulier

---

## 🎯 Application pour Petites Équipes

### Standards Essentiels

1. **Git** : Format de commit + workflow branches
2. **Code** : Structure modulaire + tests
3. **Review** : Validation par pairs
4. **Documentation** : Minimale mais présente
5. **CI/CD** : Tests automatisés

### Adaptations Recommandées

- **Simplifier** les tags de commit si nécessaire
- **Adapter** la structure aux besoins réels
- **Prioriser** les tests sur les fonctionnalités critiques
- **Documenter** les décisions importantes
- **Automatiser** ce qui peut l'être

Cette méthodologie d'Odoo SA, adaptée aux contraintes d'une petite équipe, garantit qualité, maintenabilité et évolutivité du code produit.