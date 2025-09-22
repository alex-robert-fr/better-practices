# 🏗️ Standards Code TypeScript

## 🎯 Principes

1. **TypeScript strict mode** obligatoire
2. **Architecture claire** - séparation domaine/technique
3. **Tests 80% coverage** minimum
4. **0 erreur ESLint/TypeScript**

## 📁 Structure Projet

```
src/
├── modules/
│   ├── auth/           # Domaine métier
│   │   ├── components/ # LoginForm
│   │   ├── services/   # authAPI
│   │   └── types/      # AuthUser
│   └── ui/            # Composants techniques
│       ├── Button/
│       └── Modal/
└── lib/
    ├── api.ts         # Config API
    └── config.ts      # Variables env
```

**Règles :**
- Un module = un domaine métier OU technique
- Export centralisé via `index.ts` optionnel
- Lazy loading uniquement pour modules lourds

## 🏷️ Conventions Nommage

```typescript
// Variables/fonctions : camelCase
const userName = "john";
function getUserData(): User { }

// Constantes : UPPER_SNAKE_CASE
const MAX_ATTEMPTS = 5;

// Types/Interfaces : PascalCase
interface User { id: number; name: string; }
type ApiResponse<T> = { data: T; error?: string; };

// Composants : PascalCase
export const UserProfile = () => <div />;

// Hooks : use + PascalCase
export const useUserData = (id: number) => { };

// Fichiers : kebab-case
// user-profile.tsx, auth-service.ts
```

## ⚙️ Configuration TypeScript

```json
// tsconfig.json - OBLIGATOIRE
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noUnusedLocals": true,
    "noUncheckedIndexedAccess": true
  }
}
```

## 🛡️ Types et Validation

```typescript
// Validation avec Zod
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100)
});

type User = z.infer<typeof UserSchema>;

// Result pattern pour gestion erreurs
const getUser = async (id: number): Promise<Result<User>> => {
  try {
    const user = await db.findUser(id);
    return user ? { success: true, data: user } :
                  { success: false, error: 'User not found' };
  } catch {
    return { success: false, error: 'Database error' };
  }
};
```

## 🧪 Tests

**Obligation :** 80% coverage minimum

```typescript
// __tests__/user.test.ts
describe('UserService', () => {
  it('should create user', async () => {
    const userData = { email: 'test@ex.com', name: 'Test' };
    const result = await userService.create(userData);

    expect(result.success).toBe(true);
    expect(result.data.email).toBe(userData.email);
  });
});
```

## 🔒 Sécurité

**Règles obligatoires :**
1. Aucun secret dans le code
2. Validation côté serveur systématique
3. HTTPS en production
4. JWT avec refresh tokens

## ⚡ Performance

**Next.js :**
- Dynamic imports pour code splitting
- next/image pour optimisation images
- SWR/React Query pour cache

**Nest.js :**
- Pas de requêtes N+1
- Pagination obligatoire
- Cache Redis si nécessaire

## 🎯 Métriques Minimales

- Tests coverage : ≥80%
- TypeScript errors : 0
- ESLint errors : 0
- Lighthouse score : ≥90
