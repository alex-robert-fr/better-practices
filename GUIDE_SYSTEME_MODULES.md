# Guide du Système de Modules avec Hooks

## Concept Principal

Imagine ton application comme une **ville** :
- Chaque module est un **quartier indépendant** (Users, Analytics, Notifications...)
- Chaque quartier a ses **prises électriques** (hooks) pour recevoir de l'électricité
- D'autres quartiers peuvent **brancher leurs câbles** sur ces prises pour ajouter des services
- Si un quartier a un problème, les autres continuent de fonctionner normalement

## Comment ça Fonctionne Concrètement

### 1. Le Module Principal (ex: Users)

Le module Users crée et gère les utilisateurs. Pendant son travail, il offre des **points de branchement** :

```typescript
// Quand je crée un utilisateur, voici où d'autres peuvent se brancher :
async createUser(data) {
  // 🔌 POINT 1: "Quelqu'un veut valider avant moi ?"
  await this.hooks.executeHooks('user.beforeValidation', data);

  // Ma validation normale
  const isValid = this.validateUser(data);

  // 🔌 POINT 2: "Quelqu'un veut faire quelque chose après validation ?"
  await this.hooks.executeHooks('user.afterValidation', data);

  // Ma création normale
  const user = this.createInDatabase(data);

  // 🔌 POINT 3: "Quelqu'un veut faire quelque chose avec ce nouvel utilisateur ?"
  await this.hooks.executeHooks('user.afterCreate', user);

  return user;
}
```

**Résultat** : Le module Users fait son travail normalement, mais offre 3 moments où d'autres peuvent "intervenir".

### 2. Le Module d'Extension (ex: Analytics)

Le module Analytics veut **tracker les créations d'utilisateurs**. Au lieu de modifier Users, il se "branche" :

```typescript
// Installation du module Analytics
async install() {
  // Je trouve le module Users
  const userModule = this.findModule('users');

  // Je me branche sur son point "afterCreate"
  userModule.hooks.addHook('user.afterCreate', (user) => {
    // Mon code Analytics s'exécute automatiquement !
    this.trackEvent('user_created', { userId: user.id });
  });
}
```

**Résultat** : Maintenant, chaque fois que Users crée un utilisateur, Analytics tracke automatiquement !

### 3. Le Registry (Tableau de Bord Central)

Le Registry sait quels modules existent et gère leur installation :

```typescript
// Au démarrage de l'app
ModuleRegistry.register(new UsersModule());     // D'abord Users
ModuleRegistry.register(new AnalyticsModule()); // Puis Analytics (qui se branche)
ModuleRegistry.register(new EmailModule());     // Puis Email (qui se branche aussi)
```

## Exemple Complet Pas à Pas

### Scénario : Système E-commerce

**1. Module Products (base)**
```typescript
class ProductModule {
  async createProduct(data) {
    // Validation locale
    await this.validateProduct(data);

    // 🔌 Autres modules peuvent ajouter des validations
    await this.hooks.executeHooks('product.beforeCreate', data);

    // Création en base
    const product = await this.productService.create(data);

    // 🔌 Autres modules peuvent réagir à la création
    await this.hooks.executeHooks('product.afterCreate', product);

    return product;
  }
}
```

**2. Module Inventory (extension)**
```typescript
class InventoryModule {
  async install() {
    const productModule = this.findModule('products');

    // Je me branche pour vérifier le stock avant création
    productModule.hooks.addHook('product.beforeCreate', (data) => {
      return this.checkStockAvailability(data);
    });

    // Je me branche pour réserver le stock après création
    productModule.hooks.addHook('product.afterCreate', (product) => {
      return this.reserveStock(product.id, product.quantity);
    });
  }
}
```

**3. Module Notifications (extension)**
```typescript
class NotificationModule {
  async install() {
    const productModule = this.findModule('products');

    // Je me branche pour notifier les admins
    productModule.hooks.addHook('product.afterCreate', (product) => {
      return this.sendEmail('admin@shop.com', `Nouveau produit: ${product.name}`);
    });
  }
}
```

### Résultat Final

Quand tu appelles `productModule.createProduct(data)` :

1. **Products** valide le produit normalement
2. **Inventory** vérifie automatiquement le stock (hook beforeCreate)
3. **Products** crée le produit en base
4. **Inventory** réserve automatiquement le stock (hook afterCreate)
5. **Notifications** envoie automatiquement l'email (hook afterCreate)

**Et le plus beau** : Products n'a aucune idée qu'Inventory et Notifications existent !

## Structure Simple Recommandée

```
src/modules/
├── products/
│   ├── product.module.ts     // Définit les hooks + logique métier
│   └── product.service.ts    // Logique pure sans hooks
├── inventory/
│   ├── inventory.module.ts   // Se branche sur products
│   └── inventory.service.ts
└── notifications/
    ├── notification.module.ts // Se branche sur products
    └── notification.service.ts
```

## Règles d'Or

### 1. Un Module = Une Responsabilité
- **Products** : gère uniquement les produits
- **Inventory** : gère uniquement les stocks
- **Notifications** : gère uniquement les emails

### 2. Extension par Hooks Uniquement
```typescript
// ❌ INTERDIT : Modifier directement un autre module
products.service.ts → inventory.check()

// ✅ CORRECT : Se brancher via hooks
inventory.module.ts → products.hooks.addHook()
```

### 3. Gestion d'Erreurs Propre
```typescript
// Chaque hook peut échouer sans casser les autres
productModule.hooks.addHook('product.afterCreate', (product) => {
  // Si l'email échoue, le produit est quand même créé
  return this.sendEmail(...).mapErr(err =>
    console.log('Email failed but product created:', err)
  );
});
```

## Tests Simplifiés

### Test d'un Module Seul
```typescript
// Test Products sans les autres modules
it('should create product', () => {
  const productModule = new ProductModule();
  const result = productModule.createProduct(mockData);
  expect(result.isOk()).toBe(true);
});
```

### Test d'Extension
```typescript
// Test que Inventory se branche bien
it('should check stock when product created', () => {
  const productModule = new ProductModule();
  const inventoryModule = new InventoryModule();

  // Installation (branchement)
  inventoryModule.install();

  // Création produit
  productModule.createProduct(mockData);

  // Vérifier que le stock a été vérifié
  expect(inventoryService.checkStock).toHaveBeenCalled();
});
```

## Avantages Concrets

1. **Développement Parallèle** : 3 devs peuvent travailler sur 3 modules différents
2. **Zéro Regression** : Modifier Inventory ne peut pas casser Products
3. **Features Optionnelles** : Désactiver Notifications = app fonctionne quand même
4. **Évolutivité** : Ajouter module Promotions sans toucher à l'existant

C'est exactement le principe d'Odoo : modules indépendants + hooks pour extension !