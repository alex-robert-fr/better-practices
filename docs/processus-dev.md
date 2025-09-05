# 🔄 Processus de Développement - Workflow Odoo SA

**Processus INTRANSIGEANT basé sur les méthodes d'Odoo SA**

---

## 🎯 Philosophie du Processus

> **"Quality is not negotiable, Speed comes from Process"** - Odoo SA

Odoo SA traite +50,000 commits par an avec **ZÉRO compromis** sur la qualité. Notre processus s'en inspire.

---

## 📋 Vue d'Ensemble du Workflow

```mermaid
flowchart TD
    A[📝 Issue/Feature Request] --> B[🎯 Planning & Analysis]
    B --> C[🌿 Create Feature Branch]
    C --> D[💻 Development + Tests]
    D --> E[🔍 Self Review + Quality Check]
    E --> F[📤 Pull Request]
    F --> G[👥 Code Review]
    G --> H{Review OK?}
    H -->|❌ Changes Needed| I[🔧 Address Feedback]
    I --> G
    H -->|✅ Approved| J[🧪 CI/CD Pipeline]
    J --> K{Tests Pass?}
    K -->|❌ Failed| L[🐛 Fix Issues]
    L --> J
    K -->|✅ All Green| M[🚀 Merge to Main]
    M --> N[📦 Deploy to Staging]
    N --> O[✅ QA Validation]
    O --> P[🌐 Deploy to Production]
```

---

## 🎯 Phase 1 : Planning & Analysis (OBLIGATOIRE)

### Template de Planification

```markdown
# 📋 Analyse Fonctionnelle - [FEATURE_NAME]

## 🎯 Objectif Métier
**Problème à résoudre :** 
[Description claire du problème business]

**Valeur apportée :**
[Impact métier mesurable]

**Critères de succès :**
- [ ] Critère 1 (métrique précise)
- [ ] Critère 2 (métrique précise)
- [ ] Critère 3 (métrique précise)

## 🏗️ Analyse Technique

### Architecture Impact
- **Modules impactés :** 
- **Dépendances :** 
- **Breaking changes :** Oui/Non
- **Migration nécessaire :** Oui/Non

### Estimation
- **Complexité :** Simple/Medium/Complex
- **Effort :** [X] jours développeur
- **Tests :** [X] jours
- **Documentation :** [X] heures

### Risques Identifiés
| Risque | Impact | Probabilité | Mitigation |
|--------|--------|-------------|------------|
| Risk 1 | H/M/L  | H/M/L       | Action     |

## ✅ Definition of Done
- [ ] Code implémenté selon standards
- [ ] Tests unitaires >80% couverture  
- [ ] Tests d'intégration complets
- [ ] Documentation mise à jour
- [ ] Code review approuvé
- [ ] CI/CD pipeline vert
- [ ] QA validation OK
- [ ] Performance benchmarks OK
```

### Checklist Pre-Development

```bash
#!/bin/bash
# scripts/pre-dev-check.sh

echo "🔍 Vérification pré-développement..."

# 1. Branch principale à jour
git fetch origin
if [ $(git rev-list HEAD..origin/main --count) -ne 0 ]; then
    echo "❌ Branch main pas à jour. Faites: git pull origin main"
    exit 1
fi

# 2. Environnement clean
if [ -n "$(git status --porcelain)" ]; then
    echo "❌ Workspace pas clean. Commitez ou stashez vos changements"
    exit 1
fi

# 3. Tests passent
pytest || {
    echo "❌ Tests échouent sur main. Résolvez avant de commencer"
    exit 1
}

echo "✅ Prêt pour développement"
```

---

## 🌿 Phase 2 : Feature Branch Creation

### Convention de Nommage STRICTE

```bash
# Format OBLIGATOIRE
feature/[module]-[description-courte]
bugfix/[module]-[bug-description]  
hotfix/[critical-issue]
refactor/[module]-[refactor-scope]

# ✅ Exemples corrects
feature/auth-password-policy
feature/inventory-barcode-scanner
bugfix/payment-gateway-timeout
hotfix/security-user-bypass
refactor/database-query-optimization

# ❌ Interdits
feature/new-stuff
fix-bug
working-on-something
temp-branch
```

### Création de Branch Standard

```bash
#!/bin/bash
# scripts/create-feature-branch.sh

if [ $# -ne 2 ]; then
    echo "Usage: $0 <type> <description>"
    echo "Types: feature|bugfix|hotfix|refactor"
    echo "Example: $0 feature auth-password-policy"
    exit 1
fi

TYPE=$1
DESCRIPTION=$2
BRANCH_NAME="${TYPE}/${DESCRIPTION}"

echo "🌿 Création de la branche: ${BRANCH_NAME}"

# Vérification format
if ! [[ "$DESCRIPTION" =~ ^[a-z0-9-]+$ ]]; then
    echo "❌ Description doit être en lowercase avec tirets uniquement"
    exit 1
fi

# Création depuis main à jour
git checkout main
git pull origin main
git checkout -b "$BRANCH_NAME"

echo "✅ Branche créée: ${BRANCH_NAME}"
echo "📝 N'oubliez pas de:"
echo "   1. Créer issue GitHub liée"
echo "   2. Planifier les tests"
echo "   3. Définir les critères de succès"
```

---

## 💻 Phase 3 : Développement STRICT

### Cycle de Développement TDD

```python
# 1. RED - Écrire test qui échoue
def test_user_password_validation_enforces_policy():
    # Arrange
    weak_password = "123456"
    
    # Act & Assert
    with pytest.raises(WeakPasswordError) as exc_info:
        user_service.create_user("test@example.com", "Test User", weak_password)
    
    assert "Password must contain at least 12 characters" in str(exc_info.value)

# 2. GREEN - Code minimal qui passe
class UserService:
    def create_user(self, email: str, name: str, password: str) -> User:
        if len(password) < 12:
            raise WeakPasswordError("Password must contain at least 12 characters")
        
        # Minimal implementation
        return User(email, name, password)

# 3. REFACTOR - Améliorer le code
class PasswordPolicy:
    def __init__(self):
        self.min_length = 12
        self.require_uppercase = True
        self.require_lowercase = True
        self.require_digits = True
        self.require_special_chars = True
    
    def validate(self, password: str) -> None:
        if len(password) < self.min_length:
            raise WeakPasswordError(f"Password must be at least {self.min_length} characters")
        
        if self.require_uppercase and not re.search(r'[A-Z]', password):
            raise WeakPasswordError("Password must contain uppercase letters")
        
        # ... autres validations

class UserService:
    def __init__(self, password_policy: PasswordPolicy):
        self._password_policy = password_policy
    
    def create_user(self, email: str, name: str, password: str) -> User:
        self._password_policy.validate(password)
        return User(email, name, password)
```

### Commits Atomiques OBLIGATOIRES

```bash
# ✅ Commit progression logique
git add tests/test_password_policy.py
git commit -m "[TEST] auth: add password policy validation tests

Add comprehensive test suite for password strength validation
covering minimum length, character requirements, and edge cases.
Tests ensure business requirements for enterprise security."

git add src/core/services/password_policy.py
git commit -m "[ADD] auth: implement configurable password policy

Add PasswordPolicy class with configurable validation rules:
- Minimum length (default 12 characters)
- Character type requirements (upper, lower, digits, special)
- Extensible for future policy additions

Supports enterprise security compliance requirements."

git add src/core/services/user_service.py
git commit -m "[IMP] auth: integrate password policy in user creation

Enhance user creation workflow with password validation:
- Validate password strength before user creation
- Provide clear error messages for policy violations
- Maintain backward compatibility with existing API"
```

### Tests Obligatoires par Feature

```python
# tests/unit/core/services/test_user_service.py
class TestUserService:
    """Tests unitaires service utilisateur"""
    
    @pytest.fixture
    def user_service(self):
        password_policy = PasswordPolicy()
        user_repo = Mock()
        return UserService(password_policy, user_repo)
    
    def test_create_user_with_valid_password_succeeds(self, user_service):
        # Arrange
        email = "test@example.com"
        name = "Test User"  
        strong_password = "SecurePass123!@#"
        
        # Act
        user = user_service.create_user(email, name, strong_password)
        
        # Assert
        assert user.email == email
        assert user.name == name
        assert user.password_hash is not None

# tests/integration/test_user_workflow.py  
class TestUserWorkflow:
    """Tests d'intégration workflow complet"""
    
    def test_complete_user_registration_workflow(self, test_client, test_db):
        # 1. Création utilisateur
        user_data = {
            "email": "integration@test.com",
            "name": "Integration Test",
            "password": "SecurePass123!@#"
        }
        
        response = test_client.post("/users", json=user_data)
        assert response.status_code == 201
        
        # 2. Vérification en base
        user_id = response.json()["id"]
        saved_user = test_db.query(User).filter(User.id == user_id).first()
        assert saved_user is not None
        assert saved_user.email == user_data["email"]

# tests/performance/test_user_performance.py
class TestUserPerformance:
    """Tests de performance"""
    
    def test_user_creation_performance_under_100ms(self, user_service):
        start = time.time()
        
        user = user_service.create_user(
            "perf@test.com", 
            "Perf Test", 
            "SecurePass123!@#"
        )
        
        duration = time.time() - start
        assert duration < 0.1  # < 100ms
        assert user is not None
```

---

## 🔍 Phase 4 : Quality Gate INTRANSIGEANT

### Script de Validation Pre-PR

```bash
#!/bin/bash
# scripts/quality-gate.sh

set -e

echo "🛡️  QUALITY GATE - Standards Odoo SA"
echo "=================================="

# 1. Format Code (AUTO-FIX)
echo "📝 Auto-formatage du code..."
black src tests
isort src tests
echo "✅ Code formaté"

# 2. Linting STRICT
echo "🔍 Analyse qualité code..."
flake8 src tests --max-complexity=10 || {
    echo "❌ BLOQUÉ: Erreurs de linting détectées"
    echo "Corrigez TOUS les problèmes avant de continuer"
    exit 1
}

# 3. Type Checking STRICT
echo "🏷️  Vérification types..."
mypy src --strict || {
    echo "❌ BLOQUÉ: Erreurs de typage détectées"
    echo "Type hints OBLIGATOIRES sur toutes les fonctions"
    exit 1
}

# 4. Security Scan ZERO TOLERANCE
echo "🔒 Scan sécurité..."
bandit -r src -ll || {
    echo "❌ BLOQUÉ: Problèmes de sécurité détectés"
    echo "AUCUN problème de sécurité toléré"
    exit 1
}

# 5. Tests Coverage MINIMUM 80%
echo "🧪 Exécution tests + couverture..."
pytest --cov=src --cov-report=term --cov-fail-under=80 || {
    echo "❌ BLOQUÉ: Tests échoués ou couverture < 80%"
    echo "Ajoutez des tests pour atteindre le minimum requis"
    exit 1
}

# 6. Performance Benchmarks
echo "⚡ Tests de performance..."
pytest tests/performance/ || {
    echo "❌ BLOQUÉ: Tests de performance échoués"
    echo "Optimisez les performances avant merge"
    exit 1
}

# 7. Documentation Updated
echo "📚 Vérification documentation..."
if [ -n "$(git diff --name-only HEAD..origin/main | grep '^src/' | head -1)" ]; then
    if [ -z "$(git diff --name-only HEAD..origin/main | grep '\.md$')" ]; then
        echo "⚠️  ATTENTION: Code modifié mais pas de docs"
        echo "Pensez à mettre à jour la documentation si nécessaire"
    fi
fi

echo ""
echo "🎉 QUALITY GATE PASSED - Prêt pour Pull Request"
echo "Standards Odoo SA respectés ✅"
```

### Métriques Qualité Automatiques

```python
# scripts/generate_metrics.py
import subprocess
import json
import os
from datetime import datetime
from pathlib import Path

class QualityMetrics:
    
    def __init__(self):
        self.project_root = Path.cwd()
        self.report_dir = self.project_root / "quality-reports"
        self.report_dir.mkdir(exist_ok=True)
    
    def generate_coverage_report(self) -> dict:
        """Génère rapport de couverture détaillé"""
        result = subprocess.run(
            ['pytest', '--cov=src', '--cov-report=json', '--cov-report=html'],
            capture_output=True, text=True, check=True
        )
        
        with open('coverage.json') as f:
            coverage_data = json.load(f)
        
        return {
            'total_coverage': coverage_data['totals']['percent_covered'],
            'missing_lines': coverage_data['totals']['missing_lines'],
            'files_below_threshold': [
                file for file, data in coverage_data['files'].items() 
                if data['summary']['percent_covered'] < 80
            ]
        }
    
    def generate_complexity_report(self) -> dict:
        """Analyse complexité cyclomatique"""
        result = subprocess.run(
            ['radon', 'cc', 'src', '--json', '--min', 'C'],
            capture_output=True, text=True, check=True
        )
        
        complexity_data = json.loads(result.stdout)
        
        high_complexity_functions = []
        for file_path, functions in complexity_data.items():
            for func in functions:
                if func['complexity'] > 10:
                    high_complexity_functions.append({
                        'file': file_path,
                        'function': func['name'],
                        'complexity': func['complexity'],
                        'line': func['lineno']
                    })
        
        return {
            'high_complexity_count': len(high_complexity_functions),
            'high_complexity_functions': high_complexity_functions
        }
    
    def generate_technical_debt_report(self) -> dict:
        """Calcule la dette technique"""
        # TODO comments
        todo_result = subprocess.run(
            ['grep', '-r', '--include=*.py', 'TODO', 'src'],
            capture_output=True, text=True
        )
        todo_count = len(todo_result.stdout.splitlines()) if todo_result.returncode == 0 else 0
        
        # FIXME comments  
        fixme_result = subprocess.run(
            ['grep', '-r', '--include=*.py', 'FIXME', 'src'],
            capture_output=True, text=True
        )
        fixme_count = len(fixme_result.stdout.splitlines()) if fixme_result.returncode == 0 else 0
        
        # Duplicated code
        duplicate_result = subprocess.run(
            ['python', '-m', 'pycodestyle', '--statistics', 'src'],
            capture_output=True, text=True
        )
        
        return {
            'todo_count': todo_count,
            'fixme_count': fixme_count,
            'total_debt_markers': todo_count + fixme_count,
            'debt_level': 'HIGH' if (todo_count + fixme_count) > 10 else 'MEDIUM' if (todo_count + fixme_count) > 5 else 'LOW'
        }
    
    def generate_quality_score(self) -> dict:
        """Score qualité global (0-100)"""
        coverage = self.generate_coverage_report()
        complexity = self.generate_complexity_report()
        debt = self.generate_technical_debt_report()
        
        # Score calculation (Odoo-inspired)
        coverage_score = min(coverage['total_coverage'], 100)  # Max 100
        complexity_penalty = min(complexity['high_complexity_count'] * 5, 30)  # Max -30
        debt_penalty = min(debt['total_debt_markers'] * 2, 20)  # Max -20
        
        final_score = max(0, coverage_score - complexity_penalty - debt_penalty)
        
        quality_grade = (
            'A' if final_score >= 90 else
            'B' if final_score >= 80 else  
            'C' if final_score >= 70 else
            'D' if final_score >= 60 else 'F'
        )
        
        return {
            'score': final_score,
            'grade': quality_grade,
            'coverage_score': coverage_score,
            'complexity_penalty': complexity_penalty,
            'debt_penalty': debt_penalty,
            'details': {
                'coverage': coverage,
                'complexity': complexity,
                'technical_debt': debt
            }
        }
    
    def save_report(self, report: dict, filename: str):
        """Sauvegarde rapport avec timestamp"""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        report_file = self.report_dir / f"{filename}_{timestamp}.json"
        
        with open(report_file, 'w') as f:
            json.dump(report, f, indent=2, default=str)
        
        print(f"📊 Rapport sauvé: {report_file}")

if __name__ == "__main__":
    metrics = QualityMetrics()
    
    print("📊 Génération rapport qualité...")
    quality_report = metrics.generate_quality_score()
    metrics.save_report(quality_report, "quality_score")
    
    print(f"\n🏆 SCORE QUALITÉ: {quality_report['score']}/100 (Grade {quality_report['grade']})")
    print(f"📈 Couverture: {quality_report['details']['coverage']['total_coverage']:.1f}%")
    print(f"🔥 Complexité haute: {quality_report['details']['complexity']['high_complexity_count']} fonctions")
    print(f"⚠️  Dette technique: {quality_report['details']['technical_debt']['total_debt_markers']} marqueurs")
    
    if quality_report['grade'] in ['D', 'F']:
        print("\n❌ QUALITÉ INSUFFISANTE - Amélioration requise avant merge")
        exit(1)
    elif quality_report['grade'] == 'C':
        print("\n⚠️  QUALITÉ ACCEPTABLE - Amélioration recommandée")
    else:
        print("\n✅ EXCELLENTE QUALITÉ - Standards Odoo SA respectés")
```

---

## 📤 Phase 5 : Pull Request STRICT

### Template PR Obligatoire

```markdown
## 🎯 Type de Changement
- [ ] [ADD] Nouvelle fonctionnalité
- [ ] [FIX] Correction de bug
- [ ] [IMP] Amélioration existante  
- [ ] [REF] Refactoring
- [ ] [TEST] Tests seulement
- [ ] [DOC] Documentation seulement

## 📝 Description

### Problème Résolu
[Description claire du problème business/technique résolu]

### Solution Implémentée
[Explication technique de la solution choisie et pourquoi]

### Impact et Changements
[Modules/APIs/comportements impactés]

**⚠️ Breaking Changes:** Oui/Non
[Si oui, détailler la migration nécessaire]

## 🧪 Tests et Validation

### Coverage Report
```bash
# Coller résultat de: pytest --cov=src --cov-report=term
```

### Tests Manuels Effectués
- [ ] Test 1: [Scénario testé]
- [ ] Test 2: [Scénario testé]  
- [ ] Test 3: [Scénario testé]

### Performance Impact
[Résultats des benchmarks si applicable]

## ✅ Checklist OBLIGATOIRE

### Qualité Code
- [ ] `./scripts/quality-gate.sh` passe ✅
- [ ] Coverage >= 80% sur nouveau code
- [ ] Aucun warning linting/typing
- [ ] Performance acceptable (si applicable)

### Standards Odoo SA
- [ ] Messages de commit respectent format `[TAG] module: description`
- [ ] Un commit = une modification logique
- [ ] Code suit les conventions architecture
- [ ] Documentation mise à jour si nécessaire

### Tests
- [ ] Tests unitaires ajoutés/mis à jour
- [ ] Tests d'intégration si nécessaire
- [ ] Tests de performance si fonctionnalité critique
- [ ] Aucune régression sur tests existants

### Review
- [ ] Auto-review effectué (own code review)
- [ ] TODO/FIXME nettoyés ou justifiés
- [ ] Pas de code commenté inutile
- [ ] Logs/debug code supprimés

## 📊 Métriques Qualité

### Score Qualité Actuel
[Coller résultat de: python scripts/generate_metrics.py]

### Comparaison Pre/Post
| Métrique | Avant | Après | Δ |
|----------|-------|-------|---|
| Coverage | X%    | Y%    | +Z% |
| Complexité | X    | Y     | ±Z |
| Dette Tech | X    | Y     | ±Z |

## 🔗 Liens Connexes
- Issue: #XXX
- Documentation: [lien si applicable]
- Tickets connexes: #XXX, #YYY

---

## 📋 Pour les Reviewers

### Points d'Attention Particuliers
[Zones spécifiques nécessitant attention]

### Impact Environnements
- [ ] Développement: Testé ✅
- [ ] Staging: À déployer
- [ ] Production: Migration nécessaire? Oui/Non

**⏱️ Estimation Review:** [X] heures
```

### Critères de Review NON-NÉGOCIABLES

```python
# scripts/review-checklist.py
"""
Checklist automatique de review - Standards Odoo SA
À exécuter AVANT approbation de PR
"""

import subprocess
import sys
from pathlib import Path

class ReviewChecklist:
    
    def check_commit_format(self) -> bool:
        """Vérifie format des messages de commit"""
        result = subprocess.run(
            ['git', 'log', '--oneline', 'origin/main..HEAD'],
            capture_output=True, text=True
        )
        
        commit_messages = result.stdout.strip().split('\n')
        invalid_commits = []
        
        for commit in commit_messages:
            if commit:  # Skip empty lines
                message = commit.split(' ', 1)[1] if ' ' in commit else commit
                if not self._is_valid_commit_format(message):
                    invalid_commits.append(message)
        
        if invalid_commits:
            print("❌ COMMIT FORMAT INVALIDE:")
            for commit in invalid_commits:
                print(f"   • {commit}")
            print("\nFormat requis: [TAG] module: description")
            return False
        
        print("✅ Format commits valide")
        return True
    
    def _is_valid_commit_format(self, message: str) -> bool:
        import re
        pattern = r'^\[(ADD|FIX|IMP|REF|REM|MERGE|TEST|DOC|I18N|CONF)\] [a-z_]+: .{10,49}$'
        return bool(re.match(pattern, message))
    
    def check_code_quality(self) -> bool:
        """Vérifie qualité du code ajouté"""
        try:
            # Linting
            subprocess.run(['flake8', 'src', 'tests'], check=True, 
                         capture_output=True)
            
            # Type checking
            subprocess.run(['mypy', 'src'], check=True, 
                         capture_output=True)
            
            # Security
            subprocess.run(['bandit', '-r', 'src'], check=True, 
                         capture_output=True)
            
            print("✅ Qualité code validée")
            return True
            
        except subprocess.CalledProcessError as e:
            print(f"❌ PROBLÈME QUALITÉ CODE: {e}")
            return False
    
    def check_test_coverage(self) -> bool:
        """Vérifie couverture de tests"""
        try:
            result = subprocess.run(
                ['pytest', '--cov=src', '--cov-report=json', '--cov-fail-under=80'],
                check=True, capture_output=True, text=True
            )
            
            print("✅ Couverture tests >= 80%")
            return True
            
        except subprocess.CalledProcessError:
            print("❌ COUVERTURE TESTS INSUFFISANTE (<80%)")
            return False
    
    def check_breaking_changes(self) -> bool:
        """Détecte les breaking changes potentiels"""
        result = subprocess.run(
            ['git', 'diff', '--name-only', 'origin/main..HEAD'],
            capture_output=True, text=True
        )
        
        modified_files = result.stdout.strip().split('\n')
        critical_files = [
            'src/core/interfaces/',
            'src/adapters/web/api/',
            'requirements.txt',
            'pyproject.toml'
        ]
        
        breaking_changes = []
        for file in modified_files:
            if any(critical in file for critical in critical_files):
                breaking_changes.append(file)
        
        if breaking_changes:
            print("⚠️  BREAKING CHANGES POTENTIELS:")
            for file in breaking_changes:
                print(f"   • {file}")
            print("\nVérifiez la rétrocompatibilité et documentez les migrations")
        
        return True  # Pas bloquant mais informatif
    
    def check_performance_impact(self) -> bool:
        """Vérifie l'impact performance si tests présents"""
        perf_tests = Path('tests/performance')
        if perf_tests.exists():
            try:
                subprocess.run(['pytest', 'tests/performance/'], 
                             check=True, capture_output=True)
                print("✅ Tests performance OK")
                return True
            except subprocess.CalledProcessError:
                print("❌ RÉGRESSION PERFORMANCE DÉTECTÉE")
                return False
        else:
            print("ℹ️  Pas de tests performance")
            return True
    
    def run_full_review_check(self) -> bool:
        """Exécute la checklist complète"""
        print("🔍 REVIEW CHECKLIST - Standards Odoo SA")
        print("=" * 50)
        
        checks = [
            ("Format des commits", self.check_commit_format),
            ("Qualité du code", self.check_code_quality),
            ("Couverture tests", self.check_test_coverage), 
            ("Breaking changes", self.check_breaking_changes),
            ("Impact performance", self.check_performance_impact)
        ]
        
        all_passed = True
        for check_name, check_func in checks:
            print(f"\n🔍 {check_name}...")
            if not check_func():
                all_passed = False
        
        print("\n" + "=" * 50)
        if all_passed:
            print("🎉 REVIEW CHECKLIST PASSED")
            print("✅ PR prêt pour approbation")
            return True
        else:
            print("❌ REVIEW CHECKLIST FAILED") 
            print("🔧 Corrections requises avant approbation")
            return False

if __name__ == "__main__":
    checker = ReviewChecklist()
    success = checker.run_full_review_check()
    sys.exit(0 if success else 1)
```

---

## 🤖 Phase 6 : CI/CD Pipeline AUTOMATISÉ

### GitHub Actions - Pipeline Complet

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline - Odoo SA Standards

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  PYTHON_VERSION: "3.11"
  
jobs:
  quality-gate:
    name: 🛡️ Quality Gate
    runs-on: ubuntu-latest
    
    steps:
    - name: 📥 Checkout Code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0  # Full history for proper diff
    
    - name: 🐍 Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
        cache: 'pip'
    
    - name: 📦 Install Dependencies
      run: |
        pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
    
    - name: 🔍 Code Quality Check
      run: |
        echo "🔍 Running quality gate..."
        ./scripts/quality-gate.sh
    
    - name: 📊 Generate Quality Report
      run: |
        python scripts/generate_metrics.py > quality-report.txt
        cat quality-report.txt
    
    - name: 📤 Upload Quality Report
      uses: actions/upload-artifact@v3
      with:
        name: quality-report
        path: |
          quality-report.txt
          quality-reports/
          htmlcov/
    
    - name: 💬 Comment Quality Score
      if: github.event_name == 'pull_request'
      uses: actions/github-script@v6
      with:
        script: |
          const fs = require('fs');
          const report = fs.readFileSync('quality-report.txt', 'utf8');
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: `## 📊 Rapport Qualité\n\n\`\`\`\n${report}\n\`\`\``
          });

  security-scan:
    name: 🔒 Security Scan
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: 🐍 Setup Python  
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
    
    - name: 📦 Install Security Tools
      run: |
        pip install bandit safety semgrep
    
    - name: 🔍 Bandit Security Scan
      run: |
        bandit -r src -f json -o bandit-report.json
        bandit -r src  # Human readable output
    
    - name: 🔍 Safety Dependencies Check
      run: safety check --json --output safety-report.json || true
    
    - name: 🔍 Semgrep Security Scan
      run: |
        semgrep --config=auto src --json --output=semgrep-report.json || true
    
    - name: 📤 Upload Security Reports
      uses: actions/upload-artifact@v3
      with:
        name: security-reports
        path: |
          bandit-report.json
          safety-report.json  
          semgrep-report.json

  performance-tests:
    name: ⚡ Performance Tests
    runs-on: ubuntu-latest
    if: contains(github.event.head_commit.message, '[PERF]') || github.event_name == 'pull_request'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: 🐍 Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
    
    - name: 📦 Install Dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest-benchmark
    
    - name: ⚡ Run Performance Tests
      run: |
        pytest tests/performance/ --benchmark-json=benchmark-report.json
    
    - name: 📊 Performance Report
      run: |
        python -c "
        import json
        with open('benchmark-report.json') as f:
            data = json.load(f)
        print('📊 PERFORMANCE BENCHMARKS:')
        for test in data['benchmarks']:
            name = test['name']
            mean = test['stats']['mean']
            print(f'   • {name}: {mean:.3f}s avg')
        "
    
    - name: 📤 Upload Benchmark Report
      uses: actions/upload-artifact@v3
      with:
        name: performance-report
        path: benchmark-report.json

  integration-tests:
    name: 🧪 Integration Tests
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
    - uses: actions/checkout@v4
    
    - name: 🐍 Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
    
    - name: 📦 Install Dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest-xdist  # Parallel testing
    
    - name: 🗄️ Setup Test Database
      env:
        DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
      run: |
        python scripts/setup_test_db.py
    
    - name: 🧪 Run Integration Tests
      env:
        DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
      run: |
        pytest tests/integration/ -v --tb=short -n auto
    
    - name: 📊 Test Results Summary
      if: always()
      run: |
        echo "Integration tests completed"

  deploy-staging:
    name: 🚀 Deploy to Staging
    runs-on: ubuntu-latest
    needs: [quality-gate, security-scan, integration-tests]
    if: github.ref == 'refs/heads/develop' && github.event_name == 'push'
    
    environment:
      name: staging
      url: https://staging.yourapp.com
    
    steps:
    - uses: actions/checkout@v4
    
    - name: 🚀 Deploy to Staging
      run: |
        echo "🚀 Deploying to staging environment..."
        # Your deployment script here
        ./scripts/deploy-staging.sh
    
    - name: 🔍 Health Check
      run: |
        echo "🔍 Running health checks..."
        ./scripts/health-check.sh staging
    
    - name: 📧 Notify Team
      if: always()
      run: |
        echo "📧 Notifying team of deployment status..."
        # Notification logic (Slack, email, etc.)

  deploy-production:
    name: 🌐 Deploy to Production
    runs-on: ubuntu-latest
    needs: [quality-gate, security-scan, integration-tests]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    environment:
      name: production
      url: https://yourapp.com
    
    steps:
    - uses: actions/checkout@v4
    
    - name: 🌐 Deploy to Production
      run: |
        echo "🌐 Deploying to production..."
        ./scripts/deploy-production.sh
    
    - name: 🔍 Production Health Check
      run: |
        echo "🔍 Production health check..."
        ./scripts/health-check.sh production
    
    - name: 📊 Deployment Metrics
      run: |
        echo "📊 Recording deployment metrics..."
        python scripts/record_deployment_metrics.py

  notify-deployment:
    name: 📢 Notify Deployment
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()
    
    steps:
    - name: 📢 Success Notification
      if: needs.deploy-production.result == 'success' || needs.deploy-staging.result == 'success'
      run: |
        echo "✅ Deployment successful!"
        # Send success notification
    
    - name: 🚨 Failure Notification
      if: needs.deploy-production.result == 'failure' || needs.deploy-staging.result == 'failure'
      run: |
        echo "❌ Deployment failed!"
        # Send failure notification
```

---

## 📊 Phase 7 : Monitoring & Feedback Loop

### Métriques de Développement

```python
# scripts/dev_metrics_collector.py
import json
import subprocess
from datetime import datetime, timedelta
from typing import Dict, List
import sqlite3

class DevMetricsCollector:
    """Collecteur de métriques de développement - Style Odoo SA"""
    
    def __init__(self, db_path: str = "dev_metrics.db"):
        self.db_path = db_path
        self._init_database()
    
    def _init_database(self):
        """Initialise la base de métriques"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS metrics (
                id INTEGER PRIMARY KEY,
                date TEXT NOT NULL,
                metric_type TEXT NOT NULL,
                metric_name TEXT NOT NULL, 
                value REAL NOT NULL,
                metadata TEXT
            )
        """)
        
        conn.commit()
        conn.close()
    
    def collect_git_metrics(self) -> Dict:
        """Métriques Git (à la Odoo SA)"""
        today = datetime.now().date()
        week_ago = today - timedelta(days=7)
        
        # Commits par jour
        result = subprocess.run([
            'git', 'log', '--since', str(week_ago), 
            '--pretty=format:%ad', '--date=short'
        ], capture_output=True, text=True)
        
        commits_by_day = {}
        for date_str in result.stdout.strip().split('\n'):
            if date_str:
                commits_by_day[date_str] = commits_by_day.get(date_str, 0) + 1
        
        # Commits par auteur
        result = subprocess.run([
            'git', 'shortlog', '-sn', '--since', str(week_ago)
        ], capture_output=True, text=True)
        
        commits_by_author = {}
        for line in result.stdout.strip().split('\n'):
            if line:
                parts = line.strip().split('\t')
                if len(parts) == 2:
                    count, author = parts
                    commits_by_author[author] = int(count)
        
        return {
            'commits_by_day': commits_by_day,
            'commits_by_author': commits_by_author,
            'total_commits_week': sum(commits_by_day.values())
        }
    
    def collect_quality_metrics(self) -> Dict:
        """Métriques qualité code"""
        try:
            # Coverage
            cov_result = subprocess.run([
                'pytest', '--cov=src', '--cov-report=json'
            ], capture_output=True, text=True, check=True)
            
            with open('coverage.json') as f:
                cov_data = json.load(f)
            
            coverage = cov_data['totals']['percent_covered']
            
            # Complexité
            complexity_result = subprocess.run([
                'radon', 'cc', 'src', '--json'
            ], capture_output=True, text=True, check=True)
            
            complexity_data = json.loads(complexity_result.stdout)
            high_complexity_count = sum(
                1 for file_data in complexity_data.values()
                for func in file_data
                if func['complexity'] > 10
            )
            
            return {
                'test_coverage': coverage,
                'high_complexity_functions': high_complexity_count,
                'quality_score': max(0, coverage - (high_complexity_count * 5))
            }
            
        except Exception as e:
            print(f"Erreur collecte métriques qualité: {e}")
            return {}
    
    def collect_performance_metrics(self) -> Dict:
        """Métriques performance (si benchmarks disponibles)"""
        try:
            result = subprocess.run([
                'pytest', 'tests/performance/', '--benchmark-json=bench.json'
            ], capture_output=True, text=True, check=True)
            
            with open('bench.json') as f:
                bench_data = json.load(f)
            
            benchmarks = {}
            for test in bench_data.get('benchmarks', []):
                benchmarks[test['name']] = {
                    'mean': test['stats']['mean'],
                    'stddev': test['stats']['stddev'],
                    'min': test['stats']['min'],
                    'max': test['stats']['max']
                }
            
            return {
                'benchmark_count': len(benchmarks),
                'benchmarks': benchmarks
            }
            
        except Exception:
            return {'benchmark_count': 0, 'benchmarks': {}}
    
    def store_metrics(self, metrics: Dict, metric_type: str):
        """Sauvegarde métriques en base"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        timestamp = datetime.now().isoformat()
        
        for metric_name, value in metrics.items():
            if isinstance(value, (int, float)):
                cursor.execute("""
                    INSERT INTO metrics (date, metric_type, metric_name, value, metadata)
                    VALUES (?, ?, ?, ?, ?)
                """, (timestamp, metric_type, metric_name, value, None))
            elif isinstance(value, dict):
                cursor.execute("""
                    INSERT INTO metrics (date, metric_type, metric_name, value, metadata) 
                    VALUES (?, ?, ?, ?, ?)
                """, (timestamp, metric_type, metric_name, 0, json.dumps(value)))
        
        conn.commit()
        conn.close()
    
    def generate_daily_report(self) -> Dict:
        """Rapport quotidien style Odoo SA"""
        git_metrics = self.collect_git_metrics()
        quality_metrics = self.collect_quality_metrics()
        perf_metrics = self.collect_performance_metrics()
        
        # Stockage en base
        self.store_metrics(git_metrics, 'git')
        self.store_metrics(quality_metrics, 'quality')
        self.store_metrics(perf_metrics, 'performance')
        
        report = {
            'date': datetime.now().isoformat(),
            'git': git_metrics,
            'quality': quality_metrics,
            'performance': perf_metrics,
            'summary': {
                'commits_today': git_metrics.get('total_commits_week', 0),
                'quality_score': quality_metrics.get('quality_score', 0),
                'test_coverage': quality_metrics.get('test_coverage', 0),
                'performance_tests': perf_metrics.get('benchmark_count', 0)
            }
        }
        
        return report
    
    def get_weekly_trends(self) -> Dict:
        """Tendances hebdomadaires"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        week_ago = (datetime.now() - timedelta(days=7)).isoformat()
        
        cursor.execute("""
            SELECT metric_type, metric_name, AVG(value) as avg_value
            FROM metrics 
            WHERE date >= ? AND value != 0
            GROUP BY metric_type, metric_name
        """, (week_ago,))
        
        trends = {}
        for row in cursor.fetchall():
            metric_type, metric_name, avg_value = row
            if metric_type not in trends:
                trends[metric_type] = {}
            trends[metric_type][metric_name] = avg_value
        
        conn.close()
        return trends

if __name__ == "__main__":
    collector = DevMetricsCollector()
    
    print("📊 Génération rapport quotidien...")
    daily_report = collector.generate_daily_report()
    
    print(f"\n📈 RAPPORT DÉVELOPPEMENT - {daily_report['date'][:10]}")
    print("=" * 50)
    
    summary = daily_report['summary']
    print(f"📝 Commits semaine: {summary['commits_today']}")
    print(f"🏆 Score qualité: {summary['quality_score']:.1f}/100")
    print(f"📊 Couverture tests: {summary['test_coverage']:.1f}%")
    print(f"⚡ Tests performance: {summary['performance_tests']}")
    
    # Sauvegarde rapport
    with open(f"daily_report_{datetime.now().strftime('%Y%m%d')}.json", 'w') as f:
        json.dump(daily_report, f, indent=2)
    
    print(f"\n💾 Rapport sauvé: daily_report_{datetime.now().strftime('%Y%m%d')}.json")
```

### Dashboard Métriques

```python
# scripts/generate_dashboard.py
import json
import sqlite3
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import pandas as pd
from pathlib import Path

class MetricsDashboard:
    """Dashboard métriques style Odoo SA"""
    
    def __init__(self, db_path: str = "dev_metrics.db"):
        self.db_path = db_path
        self.output_dir = Path("dashboard")
        self.output_dir.mkdir(exist_ok=True)
    
    def generate_quality_trends_chart(self):
        """Graphique tendances qualité"""
        conn = sqlite3.connect(self.db_path)
        
        df = pd.read_sql_query("""
            SELECT date, value 
            FROM metrics 
            WHERE metric_type = 'quality' AND metric_name = 'test_coverage'
            ORDER BY date DESC LIMIT 30
        """, conn)
        
        if not df.empty:
            df['date'] = pd.to_datetime(df['date'])
            
            plt.figure(figsize=(12, 6))
            plt.plot(df['date'], df['value'], marker='o', linewidth=2)
            plt.title('📊 Évolution Couverture Tests (30 derniers jours)', fontsize=14, fontweight='bold')
            plt.xlabel('Date')
            plt.ylabel('Couverture (%)')
            plt.grid(True, alpha=0.3)
            plt.xticks(rotation=45)
            plt.tight_layout()
            
            plt.savefig(self.output_dir / 'quality_trends.png', dpi=300, bbox_inches='tight')
            plt.close()
        
        conn.close()
    
    def generate_commits_chart(self):
        """Graphique activité commits"""
        conn = sqlite3.connect(self.db_path)
        
        df = pd.read_sql_query("""
            SELECT date, value 
            FROM metrics 
            WHERE metric_type = 'git' AND metric_name = 'total_commits_week'
            ORDER BY date DESC LIMIT 30
        """, conn)
        
        if not df.empty:
            df['date'] = pd.to_datetime(df['date'])
            
            plt.figure(figsize=(12, 6))
            plt.bar(df['date'], df['value'], alpha=0.7, color='#2E86AB')
            plt.title('📝 Activité Commits (30 derniers jours)', fontsize=14, fontweight='bold')
            plt.xlabel('Date')
            plt.ylabel('Nombre de Commits')
            plt.xticks(rotation=45)
            plt.tight_layout()
            
            plt.savefig(self.output_dir / 'commits_activity.png', dpi=300, bbox_inches='tight')
            plt.close()
        
        conn.close()
    
    def generate_html_dashboard(self):
        """Dashboard HTML complet"""
        # Récupération données récentes
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # Dernières métriques
        cursor.execute("""
            SELECT metric_type, metric_name, value, date
            FROM metrics 
            WHERE date = (SELECT MAX(date) FROM metrics)
            ORDER BY metric_type, metric_name
        """)
        
        latest_metrics = {}
        for row in cursor.fetchall():
            metric_type, metric_name, value, date = row
            if metric_type not in latest_metrics:
                latest_metrics[metric_type] = {}
            latest_metrics[metric_type][metric_name] = value
        
        conn.close()
        
        html_content = f"""
<!DOCTYPE html>
<html>
<head>
    <title>📊 Dashboard Métriques - Style Odoo SA</title>
    <style>
        body {{
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            padding: 20px;
            background: #f8f9fa;
        }}
        .header {{
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 30px;
        }}
        .metrics-grid {{
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }}
        .metric-card {{
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }}
        .metric-value {{
            font-size: 2em;
            font-weight: bold;
            color: #2E86AB;
        }}
        .metric-label {{
            color: #666;
            margin-top: 5px;
        }}
        .charts-section {{
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
        }}
        .chart-container {{
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }}
        .status-good {{ color: #28a745; }}
        .status-warning {{ color: #ffc107; }}
        .status-danger {{ color: #dc3545; }}
    </style>
</head>
<body>
    <div class="header">
        <h1>📊 Dashboard Métriques de Développement</h1>
        <p>Standards Odoo SA - Dernière mise à jour: {datetime.now().strftime('%d/%m/%Y %H:%M')}</p>
    </div>
    
    <div class="metrics-grid">
        <div class="metric-card">
            <div class="metric-value status-good">
                {latest_metrics.get('quality', {}).get('test_coverage', 0):.1f}%
            </div>
            <div class="metric-label">📊 Couverture Tests</div>
        </div>
        
        <div class="metric-card">
            <div class="metric-value status-good">
                {latest_metrics.get('quality', {}).get('quality_score', 0):.0f}/100
            </div>
            <div class="metric-label">🏆 Score Qualité</div>
        </div>
        
        <div class="metric-card">
            <div class="metric-value">
                {latest_metrics.get('git', {}).get('total_commits_week', 0)}
            </div>
            <div class="metric-label">📝 Commits Semaine</div>
        </div>
        
        <div class="metric-card">
            <div class="metric-value status-warning">
                {latest_metrics.get('quality', {}).get('high_complexity_functions', 0)}
            </div>
            <div class="metric-label">🔥 Fonctions Complexes</div>
        </div>
    </div>
    
    <div class="charts-section">
        <div class="chart-container">
            <h3>📈 Tendances Qualité</h3>
            <img src="quality_trends.png" alt="Tendances qualité" style="max-width: 100%;">
        </div>
        
        <div class="chart-container">
            <h3>📝 Activité Commits</h3>
            <img src="commits_activity.png" alt="Activité commits" style="max-width: 100%;">
        </div>
    </div>
    
    <div class="metric-card" style="margin-top: 30px;">
        <h3>🎯 Objectifs Odoo SA</h3>
        <ul>
            <li>Couverture tests: <strong>≥80%</strong> ✅</li>
            <li>Score qualité: <strong>≥85/100</strong> ✅</li>
            <li>Fonctions complexes: <strong>≤5</strong> ⚠️</li>
            <li>Commits conformes: <strong>100%</strong> ✅</li>
        </ul>
    </div>
</body>
</html>
        """
        
        with open(self.output_dir / 'index.html', 'w', encoding='utf-8') as f:
            f.write(html_content)
        
        print(f"📊 Dashboard généré: {self.output_dir / 'index.html'}")

if __name__ == "__main__":
    dashboard = MetricsDashboard()
    
    print("📊 Génération dashboard métriques...")
    dashboard.generate_quality_trends_chart()
    dashboard.generate_commits_chart()  
    dashboard.generate_html_dashboard()
    
    print("✅ Dashboard complet généré!")
    print(f"🌐 Ouvrez: {dashboard.output_dir / 'index.html'}")
```

---

## 🎯 Application Immédiate - Setup Complet

### 1. Scripts de Setup (15min)

```bash
#!/bin/bash
# scripts/setup-odoo-workflow.sh

echo "🚀 Setup Workflow Odoo SA - Standards INTRANSIGEANTS"
echo "=================================================="

# 1. Structure des scripts
mkdir -p scripts hooks/.github/workflows

# 2. Hooks Git
cat > hooks/pre-commit << 'EOF'
#!/bin/bash
echo "🔍 Pre-commit Quality Gate..."
./scripts/quality-gate.sh
EOF

cat > hooks/pre-push << 'EOF'  
#!/bin/bash
echo "🔍 Pre-push Review Check..."
python scripts/review-checklist.py
EOF

chmod +x hooks/*

# 3. Installation hooks
cp hooks/* .git/hooks/

# 4. Configuration Git
git config core.hooksPath .git/hooks

# 5. Premier test
echo "✅ Workflow Odoo SA installé!"
echo "🧪 Test du quality gate..."
if ./scripts/quality-gate.sh; then
    echo "🎉 PRÊT! Standards Odoo SA activés"
else
    echo "⚠️  Corrections nécessaires avant premier commit"
fi
```

### 2. Template Premier Projet (10min)

```python
# src/core/models/__init__.py
"""
Core Business Models - Architecture Odoo SA
Modèles métier purs, sans dépendances externes
"""

# src/core/services/__init__.py  
"""
Business Services - Orchestration Odoo SA
Logique applicative et workflows métier
"""

# src/adapters/__init__.py
"""
External Adapters - Clean Architecture Odoo SA  
Interface avec systèmes externes (DB, API, etc.)
"""

# tests/unit/test_sample.py
import pytest

def test_sample_always_passes():
    """Test exemple pour valider setup"""
    assert True

class TestOdooStandards:
    """Tests validation standards Odoo SA"""
    
    def test_project_structure_exists(self):
        """Vérifie structure projet standard"""
        import os
        required_dirs = [
            'src/core/models',
            'src/core/services', 
            'src/adapters',
            'tests/unit',
            'tests/integration'
        ]
        
        for dir_path in required_dirs:
            assert os.path.exists(dir_path), f"Directory missing: {dir_path}"
    
    def test_quality_scripts_exist(self):
        """Vérifie présence scripts qualité"""
        import os
        required_scripts = [
            'scripts/quality-gate.sh',
            'scripts/review-checklist.py'
        ]
        
        for script_path in required_scripts:
            assert os.path.exists(script_path), f"Script missing: {script_path}"
```

### 3. Première Feature Standard (15min)

```bash
#!/bin/bash
# scripts/demo-first-feature.sh

echo "🎯 Démonstration première feature - Standards Odoo SA"

# 1. Création branche
git checkout -b feature/demo-user-model

# 2. Implémentation TDD
cat > tests/unit/core/test_user_model.py << 'EOF'
import pytest
from src.core.models.user import User
from src.core.exceptions import ValidationError

class TestUser:
    def test_user_creation_with_valid_data_succeeds(self):
        user = User("test@example.com", "Test User")
        assert user.email == "test@example.com"
        assert user.name == "Test User"
    
    def test_user_creation_with_invalid_email_raises_error(self):
        with pytest.raises(ValidationError, match="Email invalide"):
            User("invalid-email", "Test User")
EOF

# 3. Implémentation minimale
mkdir -p src/core/models src/core/exceptions

cat > src/core/exceptions.py << 'EOF'
class ValidationError(Exception):
    """Erreur de validation métier"""
    pass
EOF

cat > src/core/models/user.py << 'EOF'
import re
from ..exceptions import ValidationError

class User:
    def __init__(self, email: str, name: str):
        if not email or '@' not in email:
            raise ValidationError("Email invalide")
        if not name or len(name.strip()) < 2:
            raise ValidationError("Nom invalide")
        
        self.email = email
        self.name = name.strip()
EOF

# 4. Tests et commit
pytest tests/unit/core/test_user_model.py
git add tests/unit/core/test_user_model.py
git commit -m "[TEST] user: add user model validation tests

Add comprehensive tests for User model creation covering:
- Valid data scenarios with proper email and name
- Invalid email format validation with clear error messages
- Business rule validation ensuring data integrity"

git add src/core/
git commit -m "[ADD] user: implement User model with validation

Add User domain model with strict validation rules:
- Email format validation ensuring @ symbol presence  
- Name validation requiring minimum 2 characters
- Clean architecture compliance with no external dependencies

Foundation for user management system following Odoo SA patterns."

echo "✅ Première feature créée selon standards Odoo SA!"
echo "📋 Prochaines étapes:"
echo "   1. Créer PR vers develop"
echo "   2. Code review obligatoire" 
echo "   3. CI/CD pipeline validation"
echo "   4. Merge après approbation"
```

---

**🎯 RÉCAPITULATIF PROCESSUS**

| Phase | Durée | Outils | Critères Succès |
|-------|-------|---------|-----------------|
| Planning | 2h | Issue template, Analysis doc | Definition of Done claire |
| Dev Branch | 5min | Git standards | Format naming respecté |
| Development | Variable | TDD, Quality tools | Tests >80%, Standards OK |
| Quality Gate | 10min | Scripts automatisés | Tous checks verts |
| Pull Request | 30min | Template, Review | Approbation requise |
| CI/CD | 15min | GitHub Actions | Pipeline complet vert |
| Deploy | 10min | Automated scripts | Health checks OK |

**🛡️ RÈGLE ABSOLUE :** Processus NON-NÉGOCIABLE. Odoo SA refuse tout raccourci. Nous aussi.

*Un processus respecté = Code de qualité production.*