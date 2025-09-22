# 🔄 Processus Développement

## 🎯 Philosophie

> **"Qualité non-négociable, rapidité par processus"**

## 📋 Workflow

```
Issue → Branch → Dev → Tests → PR → Review → Merge → Deploy
```

## 📝 1. Issue/Planning

**Template Issue GitHub :**
```markdown
## 🎯 Type
- [ ] 🐛 Bug
- [ ] ✨ Feature
- [ ] ⚡ Improvement

## 📋 Description
**Problème/Objectif :** [Description claire]
**Solution :** [Comment résoudre]

## ✅ Critères succès
- [ ] Critère 1 (mesurable)
- [ ] Critère 2 (mesurable)

## 📊 Estimation
- Temps : X jours
- Complexité : S/M/L
- Stack : Frontend/Backend/Fullstack

## 🏷️ Labels
`type:feature` `complexity:m` `stack:frontend`
```

**Labels essentiels :**
- `type:` bug, feature, improvement
- `complexity:` s, m, l
- `stack:` frontend, backend, fullstack

## 🌿 2. Branch Creation

```bash
# Vérifications avant dev
git fetch origin
git status  # doit être clean
npm test   # doit passer

# Création branch
git checkout -b feature/auth-login
```

**Convention nommage :**
- `feature/module-description`
- `bugfix/module-probleme`
- `hotfix/probleme-urgent`

## 💻 3. Développement TDD

```typescript
// 1. RED - Test qui échoue
it('should validate user email', () => {
  expect(() => createUser('invalid')).toThrow('Email invalide');
});

// 2. GREEN - Code qui passe
const createUser = (email: string) => {
  if (!email.includes('@')) throw new Error('Email invalide');
  return { email };
};

// 3. REFACTOR - Améliorer
const createUser = (email: string) => {
  const emailSchema = z.string().email();
  const validEmail = emailSchema.parse(email);
  return { email: validEmail };
};
```

**Commits atomiques :**
```bash
# Tests d'abord
git add tests/
git commit -m "🧪 auth: ajouter tests validation email"

# Code ensuite
git add src/
git commit -m "✨ auth: implémenter validation email"
```

**Tests obligatoires :**
- Tests unitaires avant code
- Coverage ≥80%
- Tests d'intégration pour API
- Un test = un comportement

## 🔍 4. Quality Check

```bash
# Avant PR
npm run lint
npm run type-check
npm run test

# Si tout passe
echo "✅ Prêt pour PR"
```

## 📤 5. Pull Request

**Template PR :**
```
## Changes
- Change 1
- Change 2

## Tests
- [ ] Tests passent
- [ ] Coverage ≥80%

## Checklist
- [ ] Code review fait
- [ ] Pas de TODO/FIXME
```

## 👥 6. Code Review

**Critères approbation :**
1. Code lisible et maintenable
2. Tests couvrent les cas d'usage
3. Respect des standards
4. Performance acceptable

**Délai review :** 24h max

## 🚀 7. Merge & Deploy

```bash
# Après approbation PR
git checkout main
git pull origin main
git merge feature/auth-login
git push origin main

# Deploy automatique ou manuel
```

**Règles merge :**
- 1 approbation minimum
- Tests CI passent
- Pas de conflit
- Squash si commits multiples

## 🛡️ Métriques Qualité

**Obligations :**
- Tests coverage ≥80%
- 0 erreur TypeScript/ESLint
- Temps response API <500ms
- Score Lighthouse ≥90
