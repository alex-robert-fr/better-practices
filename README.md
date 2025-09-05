# 🏗️ Standards de Développement TypeScript/Next.js/Nest.js

**Méthodes intransigeantes inspirées d'Odoo, adaptées pour équipes 1-3 développeurs**

## 🎯 Philosophie

> **"Type Safety First, Code for Scale"**

Cette documentation transpose la **rigueur absolue d'Odoo** vers la stack moderne TypeScript :
- **Standards non-négociables** dès le premier commit  
- **Architecture modulaire** pensée pour grandir
- **Tests 80% coverage** obligatoires
- **Commits emojis français** explicites

**Si c'est assez bon pour Odoo, c'est notre minimum acceptable.**

## 📁 Documentation

| Document | Focus | Priorité |
|----------|--------|----------|
| **[🌿 Standards Git](docs/git-standards.md)** | Commits emojis, branches, workflow | **CRITIQUE** |
| **[🏗️ Standards Code](docs/code-standards.md)** | Architecture TS/Next/Nest, tests | **CRITIQUE** |
| **[⚙️ Processus Dev](docs/processus-dev.md)** | Review, déploiement | **IMPORTANT** |
| **[📋 Organisation Projet](docs/organisation-projet.md)** | Issues, sprints | **UTILE** |

## 🚫 Règles Intransigeantes

### ❌ INTERDIT
- Code TypeScript avec erreurs → **Commit bloqué**
- Messages commit vagues ("fix", "wip") → **Rejet automatique**
- Coverage <80% → **PR rejetée**
- `any` non justifié → **Refactor obligatoire**

### ✅ OBLIGATOIRE
- Pre-commit hooks actifs
- Messages commit français avec emojis (✨ 🐛 ♻️ ⚡ 🧪 📝 🔧)
- Tests sur nouvelle logique métier
- Code review sur features critiques

## 🎖️ Métriques Minimales

- **Coverage tests** : ≥80% global
- **Erreurs TypeScript** : 0
- **Performance Lighthouse** : ≥90 (Next.js)
- **Temps réponse API** : <500ms (Nest.js)

---

🏁 **Prêt ?** → Commence par [🌿 Standards Git](docs/git-standards.md)
