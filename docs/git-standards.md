# 🌿 Standards Git - Next.js/Nest.js STRICT

**Standards INTRANSIGEANTS inspirés des pratiques d'Odoo SA**

---

## 🎯 Principe Fondamental

> **"Un commit doit raconter une histoire claire et complète"** - Odoo SA

Chaque commit suit des règles strictes et NON-NÉGOCIABLES, même en solo. Aucune exception ne doit être faite.

---

## 📝 Format de Commit OBLIGATOIRE

### Structure Exacte
```
[TAG] scope: description courte en francais (< 50 caractères)

Description détaillée expliquant le POURQUOI du changement.
Jamais le QUOI (visible dans le diff).
```

### Validation Automatique
Le message doit former une phrase valide avec :
```
"Si appliqué, ce commit va [votre message]"
```

**✅ Correct :** `[FEAT] auth: ajouter mécanisme de refresh JWT`
**❌ Incorrect :** `[FEAT] auth: correction d'un bug`

---

## 🏷️ Tags STRICTS pour Next.js/Nest.js

### Tags Principaux
- **`✨`** - Nouvelle fonctionnalité complète
- **`🔄`** - Amélioration d'une fonctionnalité existante
- **`🐛`** - Correction de bug 
- **`♻️`** - Refactoring massif sans changement fonctionnel
- **`⚡`** - Amélioration de performance mesurable
- **`🧪`** - Tests uniquement
- **`📝`** - Documentation uniquement
- **`🔧`** - Configuration, dépendances, outils

### Scopes OBLIGATOIRES Next.js/Nest.js
```bash
# Frontend Next.js
✨ ui: ajouter composant dashboard utilisateur
🐛 api: gérer réponse vide dans service utilisateur
♻️ components: extraire logique modale réutilisable

# Backend Nest.js  
✨ auth: implémenter flow reset password
🐛 db: résoudre timeout pool connexions
⚡️ queries: optimiser recherche utilisateur avec index

# Fullstack
✨ user-profile: ajouter upload avatar
🔧 docker: mettre à jour node vers v20
🧪 integration: ajouter flow inscription utilisateur
```

---

## 🔍 Règles STRICTES (Non-négociables)

### 1. Atomicité ABSOLUE
```bash
# ✅ UN commit = UNE modification logique
git commit -m "✨ auth: implémenter login Google OAuth"

# ✅ Exception: changements liés cohérents
git commit -m "🐛 ui: corriger problèmes responsive header et sidebar"
Attention à ne pas en abuser quand même.

# ❌ JAMAIS : mélanger domaines différents  
git commit -m "✨ dashboard + 🐛 migration database"
```

### 2. Scope Modulaire OBLIGATOIRE
```bash
# ✅ Scopes par domaine métier (PRIORITAIRE)
✨ auth: intégration Google OAuth
🐛 payment: gestion webhook Stripe  
♻️ user: logique gestion profil

# ✅ Scopes techniques (ACCEPTABLE)
✨ api: endpoints utilisateur
🐛 ui: layout responsive
🔧 docker: configuration production

# ❌ Scopes vagues ou non renseigné INTERDITS
🐛 multiple: diverses corrections
✨ nouvelles fonctionnalités
```

### 3. Description POURQUOI, jamais QUOI
```bash
# ✅ Explique la logique métier (OBLIGATOIRE surtout en équipe)
🐛 auth: empêcher bypass login avec token expiré

Les tokens expirés permettaient l'accès à l'API car la validation 
middleware ne vérifiait pas l'expiration. Cela créait des failles 
de sécurité permettant l'accès non autorisé aux données utilisateur.

# ✅ Solo dev : concis mais informatif (ACCEPTABLE)
🐛 auth: corriger edge case expiration token

# ❌ Description technique INTERDITE
🐛 auth: ajouter check dans middleware
```

---

## 🌳 Workflow Branches STRICT

### Structure OBLIGATOIRE
```
main (production)
├── develop (intégration)
├── release/v2.1 (préparation release)
├── hotfix/critical-bug (corrections urgentes)
└── feature/
    ├── auth-google-oauth
    ├── dashboard-analytics-utilisateur
    └── payment-integration-stripe
```

### Règles INTRANSIGEANTES

#### Branch Principale (`main`)
- **Code de production uniquement**
- Merge uniquement depuis `release/*` ou `hotfix/*`
- **Jamais de commit direct**
- Tags de version obligatoires

#### Branch Développement (`develop`)
- Intégration des features
- Tests CI/CD systématiques
- Base pour les nouvelles features

#### Branches Features (Format STRICT)
```bash
# Nommage OBLIGATOIRE
feature/module-description-courte
feature/auth-google-oauth
feature/dashboard-analytics-utilisateur  
feature/api-gestion-payments
feature/ui-refonte-header-navigation

# Durée de vie MAXIMALE : 2 semaines
# Si plus long : DÉCOUPER obligatoirement
# Suppression IMMÉDIATE après merge
```

#### Branches Hotfix (`hotfix/*`)
```bash
# Pour corrections critiques en production
hotfix/security-user-bypass
hotfix/payment-gateway-timeout

# Merge vers main ET develop
# Tests obligatoires même en urgence
```

---

## 🔄 Workflow INTRANSIGEANT

### Solo Développement  
```bash
# Feature standard (NON-CRITIQUE)
git checkout main
git pull origin main
git checkout -b feature/amelioration-ui-dashboard

# ... développement avec commits atomiques
git add src/components/Dashboard.tsx
git commit -m "✨ ui: ajouter widgets analytics dashboard

Implémenter 3 nouveaux widgets pour métriques utilisateur :
- Widget nombre de connexions mensuelles
- Graphique évolution activité  
- Top 5 actions utilisateur populaires"

# Feature CRITIQUE ou équipe
git push origin feature/amelioration-ui-dashboard
# PR OBLIGATOIRE même solo sur features critiques
```

### Équipe 2-3 Développeurs (STRICT)
```bash
# TOUJOURS depuis develop
git checkout develop
git pull origin develop
git checkout -b feature/integration-payment-stripe

# Commits ATOMIQUES obligatoires
git add src/services/payment/
git commit -m "✨ payment: implémenter service Stripe

Ajouter service payment complet avec Stripe :
- Configuration API keys environnement
- Méthodes création/annulation subscription  
- Gestion webhooks événements payment
- Types TypeScript pour objets Stripe"

# Push + PR OBLIGATOIRE
git push origin feature/integration-payment-stripe
# Code review OBLIGATOIRE avant merge
```

---

## 🔧 Configuration OBLIGATOIRE

### Hook Pre-commit STRICT (TypeScript)
```bash
#!/bin/bash
# .git/hooks/pre-commit
echo "🔍 Validation pre-commit STRICTE..."

# 1. TypeScript check OBLIGATOIRE
if ! tsc --noEmit; then
    echo "❌ BLOQUÉ: Erreurs TypeScript détectées"
    exit 1
fi

# 2. ESLint check OBLIGATOIRE
if ! npm run lint:check; then
    echo "❌ BLOQUÉ: Erreurs ESLint détectées" 
    exit 1
fi

# 3. Format code OBLIGATOIRE
if ! npm run format:check; then
    echo "❌ BLOQUÉ: Code mal formaté (Biome)"
    exit 1
fi

# 4. Tests unitaires (si configurés)
if npm run test:unit --silent 2>/dev/null; then
    if ! npm run test:unit; then
        echo "❌ BLOQUÉ: Tests unitaires échoués"
        exit 1  
    fi
fi

echo "✅ Pre-commit validation réussie"
```

---

## 📋 PR Template STRICT

### Template OBLIGATOIRE Next.js/Nest.js
```markdown
## 🎯 Type de Changement
- [ ] ✨ **[FEAT]** - Nouvelle fonctionnalité
- [ ] 🐛 **[FIX]** - Correction de bug
- [ ] ♻️ **[REFACTOR]** - Refactoring code
- [ ] ⚡ **[PERF]** - Amélioration performance
- [ ] 🧪 **[TEST]** - Tests uniquement
- [ ] 📝 **[DOCS]** - Documentation
- [ ] 🔧 **[CONFIG]** - Configuration/outils

## 📝 Description

### Problème Résolu
[OBLIGATOIRE : Pourquoi ce changement ? Quel problème business/technique ?]

### Solution Implémentée  
[OBLIGATOIRE : Comment c'est résolu ? Choix techniques justifiés]

### Impact
[OBLIGATOIRE : Quels modules/composants/endpoints affectés ?]

## 🧪 Tests OBLIGATOIRES

### Tests Automatiques
- [ ] Tests unitaires ajoutés/mis à jour (OBLIGATOIRE si logique métier)
- [ ] Tests intégration si endpoints API (OBLIGATOIRE Nest.js)  
- [ ] Tests E2E si fonctionnalité utilisateur critique

### Tests Manuels OBLIGATOIRES
- [ ] Fonctionnalité testée localement
- [ ] Pas de régression observée
- [ ] Performance acceptable (ex: < 3s chargement page)

## 📱 Frontend Next.js (si applicable)
- [ ] Composants testés en isolation
- [ ] Pages affectées validées
- [ ] Responsive design vérifié (si nécessaire)
- [ ] SEO impact évalué (si pages publiques)
- [ ] Accessibilité respectée (WCAG basique, si nécessaire également)

## 🔧 Backend Nest.js (si applicable)
- [ ] Endpoints testés (Postman/curl)
- [ ] Validation des DTOs fonctionnelle
- [ ] Gestion erreurs implémentée
- [ ] Logs appropriés ajoutés
- [ ] Migrations base données testées

## 🐳 Infrastructure (si applicable)
- [ ] Variables environnement documentées
- [ ] Docker build successful
- [ ] Configuration production validée

## ✅ Checklist INTRANSIGEANTE

### Code Quality (NON-NÉGOCIABLE)
- [ ] TypeScript : 0 erreur, 0 `any` non justifié
- [ ] ESLint : 0 erreur, 0 warning non justifié
- [ ] Biome : Code formaté automatiquement
- [ ] Console.log supprimés (sauf logs intentionnels)
- [ ] TODOs documentés avec issue GitHub

### Standards git
- [ ] Messages commit respectent format strict
- [ ] Un commit = une modification logique atomique  
- [ ] Description PR complète et précise
- [ ] Code review OBLIGATOIRE (équipe)

### Sécurité (CRITIQUE)
- [ ] Aucun secret/credential dans le code
- [ ] Variables sensibles dans .env uniquement
- [ ] Validation input côté backend systématique
- [ ] Authentification/autorisation respectée

## 🔗 Références
- **Issue GitHub :** #XXX (OBLIGATOIRE si existe)  
- **Documentation :** [Lien si mise à jour nécessaire]
- **Dépendances :** [Issues/PRs bloquantes]

---

## ⚠️ POUR LE REVIEWER (Équipe)

### Points d'Attention OBLIGATOIRES
- [ ] Logique métier cohérente avec architecture existante
- [ ] Performance impact acceptable  
- [ ] Sécurité : pas de failles introduites
- [ ] Maintenabilité : code lisible et documenté

### Critères de Rejet AUTOMATIQUE
- ❌ Erreurs TypeScript non résolues
- ❌ Tests échoués ou couverture insuffisante
- ❌ Standards commit non respectés
- ❌ Sécurité compromise
- ❌ Performance dégradée significativement
```

---

## 🚫 INTERDICTIONS ABSOLUES

### ❌ Commits BANNIS  
```bash
# Messages vagues
git commit -m "corrections"
git commit -m "wip"
git commit -m "maj"
git commit -m "fix"

# Commits avec erreurs TS (BLOQUÉS automatiquement)
# Commits secrets/credentials
git commit -m "🔧 ajout clés API production"
```

### ❌ Pratiques INTERDITES
- **Force push sur `main`** → Blocage automatique
- **Commit direct sur `main`** → Blocage automatique
- **PR sans description** → Rejet automatique  
- **Merge commits dans les features** → Rebase obligatoire
- **Code non testé** → Rejet automatique équipe
- **Secrets dans le code** → Rejet automatique équipe
-

---

## 🔍 Validation AUTOMATIQUE

### Scripts Package.json OBLIGATOIRES
```json
{
  "scripts": {
    "type:check": "tsc --noEmit",
    "lint:check": "eslint . --ext .ts,.tsx --max-warnings 0", 
    "format:check": "prettier --check .",
    "test": "jest --passWithNoTests --coverage --coverageThreshold='{\"global\":{\"lines\":80}}'",
    "quality:check": "npm run type:check && npm run lint:check && npm run format:check && npm run test",
    "git:prepare": "npm run lint:fix && npm run format:fix && npm run type:check && npm run test",
    "git:validate": "npm run quality:check"
  }
}
```

### Workflow OBLIGATOIRE Avant Commit
```bash
# OBLIGATOIRE avant chaque commit important
npm run git:prepare

# Validation (équipe) 
npm run git:validate

# Si TOUT passe
git add .
git commit -m "✨ dashboard: implémenter analytics utilisateur avec Chart.js

Ajouter page analytics complète pour utilisateurs :
- Graphiques activité mensuelle avec Chart.js
- Métriques temps réel connexions
- Export données CSV/PDF
- Filters par période et type activité"
```

---

## 📊 Métriques STRICTES

### KPIs NON-NÉGOCIABLES
- **Conformité commits** : 100% format respecté
- **Couverture tests** : >80% nouveau code
- **Erreurs TypeScript** : 0 dans main
- **Branches actives** : <2 par dev simultanément  

### Contrôles AUTOMATIQUES
```bash
# Validation quotidienne OBLIGATOIRE
git log --oneline --since="1 day ago" | grep -v "^\[.*\]" && echo "❌ Commits non conformes détectés"

# Statistiques hebdomadaires
git log --pretty=format:"%an" --since="1 week ago" | sort | uniq -c
git log --oneline --since="1 week ago" | wc -l
```

---

## 🎯 Application IMMÉDIATE

### Setup COMPLET (20-30 min MAX)
```bash
# 1. Configuration Git OBLIGATOIRE
git config user.name "Ton Nom Complet"  
git config user.email "email@professionnel.com"
git config core.editor "code --wait"

# 2. Hook pre-commit STRICT
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
tsc --noEmit || exit 1
npm run lint:check || exit 1  
npm run format:check || exit 1
echo "✅ Validation pre-commit réussie"
EOF
chmod +x .git/hooks/pre-commit

# 3. Premier commit VALIDATION
git add .
git commit -m "🔧 git: configurer standards développement stricts

Configuration workflow Git adapté stack Next.js/Nest.js :
- Format commit obligatoire avec scopes français
- Validation TypeScript/ESLint en pre-commit
- Règles strictes inspirées Odoo SA pour équipe 1-3 dev
- Aucun compromis sur qualité code"
```

---

**🎯 RAPPEL INTRANSIGEANT :** Ces standards sont **NON-NÉGOCIABLES**. Inspirés d'Odoo SA, adaptés à la stack Next.js/Nest.js. Zéro compromis sur la qualité.

*"Si un commit ne respecte pas ces règles, il ne doit pas exister."*
