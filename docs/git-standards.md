# 🌿 Git Standards

## 🎯 Format de Commit

```bash
{gitmoji} ({scope}) {description_impérative_français}
```

### Exemples CONFORMES (comme vos commits actuels)
```bash
📝 (docs) Améliorer documentation standards git
✨ (auth) Ajouter authentification Google OAuth
🐛 (api) Corriger timeout connexion database
♻️ (ui) Refactoriser composants formulaires
```

## 🏷️ Gitmoji + Scopes STRICTS

### Gitmoji Autorisés UNIQUEMENT
- **✨** Nouvelle fonctionnalité  
- **🐛** Correction bug
- **♻️** Refactoring code
- **📝** Documentation
- **🔧** Configuration/outils

### Scopes par Projet
**Note :** Les scopes sont définis selon l'architecture et domaines métier de chaque projet.

## 🔍 Règles NON-NÉGOCIABLES

1. **Format EXACT** : `gitmoji (scope) description`
2. **Un commit = une modification logique**
3. **Description 10-49 caractères**
4. **Phrase impérative français**
5. **Aucun commit direct sur `main`**

## 🚫 INTERDICTIONS (Bloquées automatiquement)

```bash
# ❌ REJETÉ
"fix"                           # Trop vague
"✨ nouvelle feature"           # Pas de scope  
"🐛(auth)corriger bug"          # Pas d'espace
"feat: add login"               # Pas gitmoji
"✨ (invalid) Feature"          # Scope non autorisé
```

## 🌳 Workflow Branches

### Structure
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

### Branches par Type
- **`main`** - Code production uniquement, merge PR release/hotfix, jamais commit direct
- **`develop`** - Intégration features, base nouvelles features, CI/CD systématique  
- **`feature/*`** - Durée max 2 semaines, suppression après merge, format: module-description-courte
- **`release/*`** - Préparation version, merge vers main + develop
- **`hotfix/*`** - Corrections urgentes, merge vers main + develop

## 📊 Métriques NON-NÉGOCIABLES

- **Format commits:** 100% conforme
- **Erreurs TypeScript:** 0 dans `main` 
- **ESLint warnings:** 0 max
- **Coverage:** >80% nouveau code

---

**🎯 PHILOSOPHIE :**Qualité intransigeante, application réaliste**.
