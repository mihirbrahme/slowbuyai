# SlowBuy.ai Development Guidelines

## Code Standards & Best Practices

### 1. TypeScript Guidelines

```typescript
// ✅ DO: Use explicit typing
interface ProductAnalysis {
  id: string;
  url: string;
  price: number;
  recommendations: Recommendation[];
}

// ❌ DON'T: Rely on implicit any
function analyzeProduct(data) { ... }
```

#### Type Safety
- Always use explicit type annotations
- Enable strict TypeScript configuration
- Avoid using `any` type
- Use generics for reusable components
- Implement proper error types

#### Naming Conventions
- Use PascalCase for types, interfaces, and classes
- Use camelCase for variables, functions, and methods
- Use UPPER_CASE for constants
- Use descriptive, meaningful names

### 2. React/Next.js Guidelines

#### Component Structure
```typescript
// ✅ DO: Organize components logically
import { type FC } from 'react';
import { useProductAnalysis } from '@/hooks/useProductAnalysis';
import { ProductCard } from '@/components/ProductCard';

interface DashboardProps {
  userId: string;
}

export const Dashboard: FC<DashboardProps> = ({ userId }) => {
  const { products, loading, error } = useProductAnalysis(userId);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
};
```

#### State Management
- Use React Query for server state
- Use Zustand for client state
- Implement proper loading states
- Handle errors gracefully
- Use proper data fetching patterns

### 3. API Guidelines

#### RESTful Endpoints
```typescript
// ✅ DO: Follow REST conventions
// GET /api/products/:id
// POST /api/products
// PUT /api/products/:id
// DELETE /api/products/:id

// ❌ DON'T: Use non-standard endpoints
// /api/getProduct
// /api/updateProductData
```

#### Error Handling
```typescript
interface APIError {
  code: string;
  message: string;
  details?: Record<string, unknown>;
}

// Standard error response
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "The requested product could not be found",
    "details": {
      "productId": "123"
    }
  }
}
```

### 4. Testing Guidelines

#### Unit Tests
```typescript
// ✅ DO: Write comprehensive tests
describe('ProductAnalyzer', () => {
  it('should analyze product price history', async () => {
    const analyzer = new ProductAnalyzer();
    const result = await analyzer.analyzePriceHistory('product-123');
    
    expect(result).toHaveProperty('lowestPrice');
    expect(result).toHaveProperty('priceHistory');
  });
});
```

#### Integration Tests
- Test API endpoints
- Test database interactions
- Test third-party integrations
- Test error scenarios
- Test edge cases

## Development Workflow

### 1. Git Workflow

```bash
# Feature Development
git checkout -b feature/product-analysis
git add .
git commit -m "feat: implement product analysis service"
git push origin feature/product-analysis

# Bug Fixes
git checkout -b fix/notification-delay
git add .
git commit -m "fix: resolve notification delivery delay"
git push origin fix/notification-delay
```

#### Branch Naming
- `feature/*` for new features
- `fix/*` for bug fixes
- `refactor/*` for code refactoring
- `docs/*` for documentation
- `test/*` for testing improvements

#### Commit Messages
- Follow Conventional Commits
- Use descriptive messages
- Reference issues/tickets
- Keep commits atomic

### 2. Code Review Process

#### Review Checklist
- [ ] Code follows style guidelines
- [ ] Tests are included and passing
- [ ] Documentation is updated
- [ ] No security vulnerabilities
- [ ] Performance impact considered

#### Pull Request Template
```markdown
## Description
[Describe the changes made]

## Type of Change
- [ ] New feature
- [ ] Bug fix
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Code refactoring

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed

## Screenshots
[If applicable]

## Related Issues
Fixes #[issue number]
```

### 3. CI/CD Pipeline

#### Build Process
```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
```

#### Deployment Stages
1. Development
   - Automatic deployment on merge to `develop`
   - Feature branch previews
   - Integration testing

2. Staging
   - Manual promotion from development
   - User acceptance testing
   - Performance testing

3. Production
   - Manual promotion from staging
   - Canary deployments
   - Monitoring and rollback

## Performance Guidelines

### 1. Frontend Optimization

#### Code Splitting
```typescript
// ✅ DO: Use dynamic imports
const ProductAnalysis = dynamic(() => import('./ProductAnalysis'), {
  loading: () => <LoadingSpinner />
});

// ❌ DON'T: Import everything at once
import { ProductAnalysis } from './ProductAnalysis';
```

#### Image Optimization
```typescript
// ✅ DO: Use Next.js Image component
import Image from 'next/image';

<Image
  src={product.image}
  alt={product.name}
  width={300}
  height={200}
  placeholder="blur"
  loading="lazy"
/>
```

### 2. Backend Optimization

#### Caching Strategy
```typescript
// ✅ DO: Implement proper caching
const getProductAnalysis = async (productId: string) => {
  const cached = await redis.get(`product:${productId}`);
  if (cached) return JSON.parse(cached);

  const analysis = await performAnalysis(productId);
  await redis.set(`product:${productId}`, JSON.stringify(analysis), 'EX', 3600);
  return analysis;
};
```

#### Database Optimization
- Use proper indexes
- Implement query optimization
- Use connection pooling
- Implement proper pagination
- Monitor query performance

## Security Guidelines

### 1. Authentication & Authorization

```typescript
// ✅ DO: Implement proper auth checks
const protectedRoute = withAuth(async (req, res) => {
  const { user } = req;
  if (!user.hasPermission('product:analyze')) {
    return res.status(403).json({
      error: 'Insufficient permissions'
    });
  }
  // Handle request
});
```

### 2. Data Protection

```typescript
// ✅ DO: Sanitize user input
const sanitizeProductUrl = (url: string): string => {
  const sanitized = new URL(url);
  return sanitized.toString();
};

// ✅ DO: Encrypt sensitive data
const encryptUserData = (data: UserData): string => {
  return encrypt(JSON.stringify(data), process.env.ENCRYPTION_KEY);
};
```

## Documentation Guidelines

### 1. Code Documentation

```typescript
/**
 * Analyzes a product and provides purchase recommendations
 * @param productUrl - The URL of the product to analyze
 * @param userId - The ID of the user requesting the analysis
 * @returns Promise<ProductAnalysis> - The analysis results
 * @throws ProductNotFoundError - If the product cannot be found
 * @throws AnalysisError - If the analysis fails
 */
async function analyzeProduct(
  productUrl: string,
  userId: string
): Promise<ProductAnalysis> {
  // Implementation
}
```

### 2. API Documentation

```typescript
/**
 * @api {post} /api/products/analyze Analyze Product
 * @apiName AnalyzeProduct
 * @apiGroup Products
 * @apiVersion 1.0.0
 *
 * @apiParam {String} url Product URL to analyze
 * @apiParam {String} userId ID of the requesting user
 *
 * @apiSuccess {Object} analysis Product analysis results
 * @apiSuccess {String} analysis.id Analysis ID
 * @apiSuccess {String} analysis.recommendation Recommendation text
 * @apiSuccess {Number} analysis.confidence Confidence score
 *
 * @apiError (400) InvalidURL The provided URL is invalid
 * @apiError (404) ProductNotFound Product could not be found
 * @apiError (500) AnalysisError Analysis process failed
 */
```

## React Optimization Guidelines

### 1. Component Performance

#### Prevent Unnecessary Re-renders
```typescript
// ❌ DON'T: Create inline objects/functions
const ProductList = () => {
  return (
    <ProductGrid
      items={products}
      onSort={() => sortProducts()} // New function created every render
      settings={{ view: 'grid' }}   // New object created every render
    />
  );
};

// ✅ DO: Memoize callbacks and objects
const ProductList = () => {
  const handleSort = useCallback(() => {
    sortProducts();
  }, []); // Dependencies array empty if sortProducts is stable

  const settings = useMemo(() => ({
    view: 'grid'
  }), []); // Memoize object

  return (
    <ProductGrid
      items={products}
      onSort={handleSort}
      settings={settings}
    />
  );
};
```

#### Optimize Heavy Components
```typescript
// ✅ DO: Memoize expensive components
const ProductCard = memo(({ product, onAddToWishlist }: ProductCardProps) => {
  return (
    <div className="product-card">
      <ProductImage src={product.image} />
      <ProductDetails product={product} />
      <WishlistButton onClick={onAddToWishlist} />
    </div>
  );
});

// ✅ DO: Use proper dependency arrays in hooks
const useProductAnalysis = (productId: string) => {
  const [analysis, setAnalysis] = useState<Analysis | null>(null);

  useEffect(() => {
    if (!productId) return;
    
    const fetchAnalysis = async () => {
      const result = await analyzeProduct(productId);
      setAnalysis(result);
    };

    fetchAnalysis();
  }, [productId]); // Only re-run if productId changes

  return analysis;
};
```

### 2. Data Management

#### State Organization
```typescript
// ❌ DON'T: Mix local and global state
const ProductPage = () => {
  const [selectedVariant, setSelectedVariant] = useState(null);
  const [wishlistItems, setWishlistItems] = useState([]); // Should be global
  
  // Component logic...
};

// ✅ DO: Use appropriate state management
const ProductPage = () => {
  const [selectedVariant, setSelectedVariant] = useState(null);
  const { wishlistItems, addToWishlist } = useWishlistStore(); // Global state
  
  // Component logic...
};
```

#### Efficient Data Fetching
```typescript
// ✅ DO: Use React Query for server state
export const useProductData = (productId: string) => {
  return useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 30 * 60 * 1000, // 30 minutes
  });
};
```

### 3. Code Splitting

#### Route-based Splitting
```typescript
// ✅ DO: Use dynamic imports for routes
const ProductAnalysis = dynamic(() => import('@/components/ProductAnalysis'), {
  loading: () => <LoadingSpinner />,
  ssr: false // If component relies on browser APIs
});
```

#### Component-based Splitting
```typescript
// ✅ DO: Split heavy components
const ChartComponent = dynamic(() => import('@/components/Chart'), {
  loading: () => <ChartPlaceholder />,
  ssr: false
});
```

### 4. Performance Monitoring

#### React Profiler Usage
```typescript
// ✅ DO: Use Profiler for performance monitoring
import { Profiler } from 'react';

const onRenderCallback = (
  id: string,
  phase: string,
  actualDuration: number,
  baseDuration: number
) => {
  if (actualDuration > 16) { // 60fps threshold
    console.warn(`Slow render detected in ${id}`);
  }
};

const App = () => (
  <Profiler id="App" onRender={onRenderCallback}>
    <ProductDashboard />
  </Profiler>
);
```

### 5. Best Practices Checklist

1. Component Structure
   - [ ] Break down complex components
   - [ ] Use proper prop typing
   - [ ] Implement error boundaries
   - [ ] Add loading states

2. Performance Optimization
   - [ ] Memoize expensive calculations
   - [ ] Implement virtual scrolling for long lists
   - [ ] Lazy load images and components
   - [ ] Use web workers for heavy computations

3. State Management
   - [ ] Choose appropriate state location
   - [ ] Implement proper caching
   - [ ] Optimize context usage
   - [ ] Handle side effects properly

4. Testing & Monitoring
   - [ ] Add performance monitoring
   - [ ] Implement error tracking
   - [ ] Use React DevTools
   - [ ] Monitor bundle size 