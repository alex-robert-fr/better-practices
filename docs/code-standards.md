# 🏗️ Standards de Code - TypeScript/Next.js/Nest.js STRICT


## 🎯 Principes Fondamentaux

### Philosophie du Code TypeScript
> **"Type Safety First, Code for Scale"**

1. **Type Safety ABSOLU** - TypeScript strict mode obligatoire
2. **Clean Architecture** - Séparation claire domaine/infrastructure  
3. **Performance by Design** - Optimisé dès la conception
4. **Sécurité intégrée** - Validation et autorisation systématiques
5. **Testabilité** - Code conçu pour être testé facilement

---

## 📁 Architecture Modulaire ÉVOLUTIVE

#### Structure Modulaire Unifiée
```typescript
src/
├── modules/
│   ├── auth/             # Module métier (barrel export)
│   │   ├── components/   # LoginForm, AuthGuard
│   │   ├── services/     # authAPI, authService
│   │   ├── hooks/        # useAuth, useLogin
│   │   ├── types/        # AuthUser, LoginCredentials
│   │   └── index.ts      # Export centralisé du module
│   ├── reporting/        # Module métier complexe
│   │   ├── components/   # ReportDashboard, ChartComponents
│   │   ├── services/     # reportAPI, dataProcessing
│   │   ├── utils/        # Logique spécifique reporting
│   │   └── index.ts
│   ├── ui/              # Module technique (barrel export)
│   │   ├── Button/
│   │   ├── Modal/
│   │   ├── DataTable/   # Table réutilisable
│   │   └── index.ts     # Export des composants UI
│   └── common/          # Module utilitaires
│       ├── layouts/     # Layouts de page
│       ├── utils/       # Helpers génériques
│       ├── hooks/       # useLocalStorage, useApi, etc.
│       ├── types/       # Types globaux
│       └── index.ts     # Barrel optionnel
└── lib/                 # Configuration externe
    ├── api.ts          # Client API configuré
    ├── config.ts       # Variables environnement
    └── validations.ts  # Schémas Zod réutilisables
```

#### Règles de Délimitation des Modules
```typescript
// ✅ BON module métier : délimité, cohérent
modules/
└── invoice-management/
    ├── components/       # InvoiceForm, InvoiceList
    ├── services/        # invoiceAPI, invoiceService
    ├── hooks/           # useInvoices, useInvoiceForm
    └── index.ts         # Tout ce qui concerne les factures

// ✅ BON module technique : composants réutilisables
modules/
└── ui/
    ├── Button/
    ├── DataTable/       # Composant complexe réutilisable
    └── index.ts         # Design system

// ❌ MAUVAIS module : trop générique
modules/
└── utils/              # Va dans modules/common/utils/
```

#### Barrel Exports : Règles Uniformes
```typescript
// ✅ Barrel optionnel pour TOUS les modules
// modules/reporting/index.ts
export { ReportDashboard } from './components/ReportDashboard';
export { useReportData } from './hooks/useReportData';
export { reportService } from './services/reportService';
export type { ReportConfig, ReportData } from './types';

// modules/ui/index.ts  
export { Button } from './Button';
export { Modal } from './Modal';
export { DataTable } from './DataTable';

// modules/common/index.ts (maintenant autorisé)
export { formatDate } from './utils/dateUtils';
export { useLocalStorage } from './hooks/useLocalStorage';
export type { ApiResponse } from './types';

// ✅ Imports - avec barrel OU direct selon préférence
import { ReportDashboard, useReportData } from '@/modules/reporting';
import { Button, Modal } from '@/modules/ui';
import { formatDate } from '@/modules/common';
// OU import direct si préféré
import { formatDate } from '@/modules/common/utils/dateUtils';
```

#### Lazy Loading
```typescript
// Lazy loading pour modules lourds uniquement
const ReportingModule = dynamic(() => 
  import('@/modules/reporting').then(mod => ({ default: mod.ReportDashboard })), 
  { 
    ssr: false, // Si composant lourd côté client
    loading: () => <SkeletonLoader />
  }
);

// ✅ Utiliser pour :
// - Modules admin/settings
// - Composants avec dépendances lourdes (charts, editors)
// - Modules utilisés par <20% des users

// ❌ PAS utiliser pour :
// - modules/ui (composants critiques)
// - Modules légers
// - Navigation principale
```

---

## 🏷️ Conventions de Nommage STRICTES

```typescript
// Variables et fonctions : camelCase
const userName = "johnDoe";
function calculateTotalPrice(): number { return 0; }

// Constantes : UPPER_SNAKE_CASE
const MAX_LOGIN_ATTEMPTS = 5;

// Types et Interfaces : PascalCase
interface UserProfile { id: number; name: string; }
type ApiResponse<T> = { success: boolean; data?: T; error?: string; };

// Classes : PascalCase
export class UserService { /* */ }

// Enums : PascalCase avec valeurs UPPER_SNAKE_CASE
export enum UserRole { ADMIN = 'ADMIN', USER = 'USER' }

// Composants React : PascalCase
export const UserProfile: React.FC = () => <div />;

// Hooks : camelCase avec préfixe "use"
export const useUserData = (id: number) => { /* */ };

// Fichiers : kebab-case
// user-profile.component.tsx, auth-service.ts
```

---

## ⚙️ Configuration TypeScript STRICTE

### tsconfig.json - NON-NÉGOCIABLE
```json
{
  "compilerOptions": {
    "strict": true,                    // OBLIGATOIRE
    "noImplicitAny": true,            // OBLIGATOIRE  
    "strictNullChecks": true,         // OBLIGATOIRE
    "noImplicitReturns": true,        // OBLIGATOIRE
    "noUnusedLocals": true,           // OBLIGATOIRE
    "noUnusedParameters": true,       // OBLIGATOIRE
    "exactOptionalPropertyTypes": true, // OBLIGATOIRE
    "noUncheckedIndexedAccess": true  // OBLIGATOIRE
  }
}
```

---

## 📝 Documentation JSDoc OBLIGATOIRE

```typescript
/**
 * Authentifie un utilisateur avec email/mot de passe
 * 
 * @param email - Adresse email valide
 * @param password - Mot de passe (min 8 caractères)
 * @returns Promise avec résultat authentification
 * @throws {InvalidCredentialsError} Identifiants incorrects
 * @example
 * ```typescript
 * const result = await authenticateUser('user@domain.com', 'password123');
 * ```
 * @commit ✨ auth: ajouter authentification JWT
 */
export async function authenticateUser(
  email: string, 
  password: string
): Promise<AuthResult> {
  // Implementation
}
```

---

## 🛡️ Types et Validation STRICTS

### Validation avec Zod (Recommandé)
```typescript
import { z } from 'zod';

export const CreateUserRequestSchema = z.object({
  email: z.string().email('Email invalide'),
  name: z.string().min(2, 'Nom trop court').max(100, 'Nom trop long'),
  age: z.number().int().min(0).max(150).optional()
});

export type CreateUserRequest = z.infer<typeof CreateUserRequestSchema>;
```

### Gestion d'Erreurs avec Result Pattern
```typescript
export type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

export async function getUser(id: number): Promise<Result<User>> {
  try {
    const user = await userRepository.findById(id);
    if (!user) return { success: false, error: new Error('User not found') };
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}
```

---

## 🧪 Tests OBLIGATOIRES

### Structure et Couverture
- **Tests unitaires** : 80% de couverture sur tout le projet MINIMUM
- **Tests d'intégration** : API endpoints critiques  
- **Tests E2E** : Workflows utilisateur principaux

### Framework : Jest + Testing Library
```typescript
// __tests__/users.service.test.ts
describe('UserService', () => {
  it('should create user successfully', async () => {
    // Arrange
    const userData = { email: 'test@example.com', name: 'Test User' };
    
    // Act  
    const result = await userService.createUser(userData);
    
    // Assert
    expect(result.success).toBe(true);
    expect(result.data?.email).toBe(userData.email);
  });
});
```

---

## 🔧 Outils Qualité OBLIGATOIRES

### ESLint Configuration Stricte
```json
{
  "extends": ["@typescript-eslint/recommended-requiring-type-checking"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/strict-boolean-expressions": "error"
  }
}
```

### Scripts Package.json OBLIGATOIRES
```json
{
  "scripts": {
    "type:check": "tsc --noEmit",
    "lint:check": "eslint . --ext .ts,.tsx --max-warnings 0", 
    "format:check": "prettier --check .",
    "test": "jest --passWithNoTests --coverage --coverageThreshold='{\"global\":{\"lines\":80}}'",
    "quality:check": "npm run type:check && npm run lint:check && npm run format:check && npm run test"
  }
}
```

---

## 🚀 Standards API Next.js/Nest.js

### API Routes Next.js
```typescript
// pages/api/users/[id].ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<ApiResponse<User>>
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ success: false, error: 'Method not allowed' });
  }
  
  const result = await userService.getUser(parseInt(req.query.id as string));
  
  if (!result.success) {
    return res.status(404).json({ success: false, error: result.error.message });
  }
  
  return res.status(200).json({ success: true, data: result.data });
}
```

### Controllers Nest.js
```typescript
@Controller('users')
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  async createUser(@Body() createUserDto: CreateUserDto): Promise<ApiResponse<User>> {
    const result = await this.usersService.createUser(createUserDto);
    if (!result.success) {
      throw new BadRequestException(result.error.message);
    }
    return { success: true, data: result.data };
  }
}
```

---

## 🔒 Sécurité INTRANSIGEANTE

### Règles NON-NÉGOCIABLES
1. **Aucun secret** dans le code source
2. **Validation côté serveur** systématique  
3. **Sanitisation** de toutes les entrées utilisateur
4. **Authentification JWT** avec refresh tokens
5. **HTTPS** obligatoire en production
6. **Logs sécurisés** (masquage données sensibles)

---

## ⚡ Performance OBLIGATOIRE

### Next.js Optimisations
- **Dynamic imports** pour le code splitting
- **Image optimization** avec next/image
- **API routes** optimisées (pas de sur-fetch)
- **Caching** approprié (SWR/React Query)

### Nest.js Optimisations  
- **Database queries** optimisées (pas de N+1)
- **Pagination** sur toutes les listes
- **Caching** Redis pour données fréquentes
- **Compression** gzip/brotli activée

---

## 📋 Checklist Pre-Commit OBLIGATOIRE

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 1. TypeScript check
tsc --noEmit || exit 1

# 2. ESLint check  
npm run lint:check || exit 1

# 3. Format check
npm run format:check || exit 1

# 4. Tests
npm run test || exit 1

echo "✅ Pre-commit validation réussie"
```

---

## 🎯 Métriques Qualité MINIMALES

- **Couverture tests** : ≥80% sur tout le projet
- **Erreurs TypeScript** : 0  
- **Erreurs ESLint** : 0
- **Performance Lighthouse** : ≥90 (Next.js)
- **Temps réponse API** : <500ms (Nest.js)

---

## 🚀 Application Immédiate

### Setup Rapide (20-30min)
```bash
# 1. Configuration TypeScript stricte
echo '{"compilerOptions":{"strict":true,"noImplicitAny":true}}' > tsconfig.json

# 2. ESLint + Prettier
npm install --save-dev @typescript-eslint/eslint-plugin prettier

# 3. Pre-commit hook  
echo 'tsc --noEmit && npm run lint:check' > .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit

# 4. Premier commit conforme
git commit -m "🔧 config: configurer standards TypeScript stricts

Configuration initiale des standards de développement :
- TypeScript strict mode activé  
- ESLint et Prettier configurés
- Pre-commit hooks en place
- Zéro compromis sur la qualité code"
```

---

**🎯 RAPPEL INTRANSIGEANT :** Ces standards sont **NON-NÉGOCIABLES**. Inspirés d'Odoo SA, adaptés à TypeScript/Next.js/Nest.js.

*Un code qui ne respecte pas ces standards ne doit pas exister dans le repository.*
