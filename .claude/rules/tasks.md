# Implementation Task Format — _pm-plan

## Rule: Tasks break user stories into concrete, actionable work units

Each task is a focused piece of work that can be completed in one PR. Tasks are generated from acceptance criteria and grouped by layer (Backend, Frontend, Database, QC).

### File structure

```
docs/stories/story.3.2/
├── tasks/
│   ├── task-3.2.1.md           # Backend task
│   ├── task-3.2.2.md           # Database task
│   └── task-3.2.3.md           # Frontend task
└── user-story-3.2.md           # Parent story
```

### Task template

```markdown
# Task 3.2.1 — Backend: Products Module & CreateSku Endpoint

**Story:** 3.2 (Add SKU to Product Catalog)
**Layer:** Backend
**Estimated effort:** 3 hours
**Owner:** @backend-team
**Status:** Not started
**Jira:** FL-42.1 (subtask)

## Objective

Scaffold the `products` domain module in NestJS with `CreateSkuUseCase` and POST endpoint.

## What Gets Built

**New files:**
```
src/modules/products/
├── contracts/
│   ├── products.driver.ts       # IProductsDriver interface
│   └── products.tokens.ts       # DI tokens
├── drivers/
│   └── prisma-products.driver.ts
├── use-cases/
│   └── create-sku.use-case.ts   # CreateSkuUseCase
├── services/
│   └── products.service.ts      # Thin facade
├── controllers/
│   └── products.controller.ts   # HTTP handlers
├── dto/
│   ├── create-sku.dto.ts        # Zod schema + DTO
│   └── sku-response.dto.ts      # Response shape
├── products.module.ts
└── index.ts
```

**Modified files:**
- `src/app.module.ts` — register ProductsModule
- `prisma/schema.prisma` — add SKU model (see Task 3.2.2)

## Implementation Details

### CreateSkuInput (Zod schema)

```typescript
// src/modules/products/dto/create-sku.dto.ts
import { createZodDto } from 'nestjs-zod';
import { z } from 'zod';

export const CreateSkuSchema = z.object({
  name: z.string().min(1).max(255),
  category: z.enum(['PRODUCE', 'DAIRY', 'GRAINS']),
  unit: z.enum(['KG', 'CRATE', 'BOX', 'UNIT']),
  price: z.string().transform(BigInt), // Decimal via Prisma
  storageType: z.enum(['AMBIENT', 'COLD_CHAIN', 'FROZEN']),
  shelfLife: z.number().int().min(1),
});

export type CreateSkuInput = z.infer<typeof CreateSkuSchema>;
export class CreateSkuDto extends createZodDto(CreateSkuSchema) {}
```

### CreateSkuUseCase

```typescript
// src/modules/products/use-cases/create-sku.use-case.ts
import { Injectable } from '@nestjs/common';
import { PRODUCTS_DRIVER } from '../contracts/products.tokens';
import { Inject } from '@nestjs/common';
import { IProductsDriver } from '../contracts/products.driver';
import { CreateSkuInput } from '../dto/create-sku.dto';

@Injectable()
export class CreateSkuUseCase {
  constructor(
    @Inject(PRODUCTS_DRIVER) private readonly driver: IProductsDriver,
  ) {}

  async execute(input: CreateSkuInput) {
    // Check for duplicates
    const existing = await this.driver.findSkuByName(input.name, input.category);
    if (existing) {
      throw new ConflictException(`SKU name already exists in this category`);
    }

    // Create SKU
    return await this.driver.createSku(input);
  }
}
```

### ProductsController

```typescript
// src/modules/products/controllers/products.controller.ts
import {
  Controller,
  Post,
  Body,
  ApiBearerAuth,
  ApiTags,
  ApiOperation,
  ApiResponse,
} from '@nestjs/common';
import { CreateSkuUseCase } from '../use-cases/create-sku.use-case';
import { CreateSkuDto } from '../dto/create-sku.dto';
import { PRODUCTS_SERVICE } from '../contracts/products.tokens';
import { Inject } from '@nestjs/common';
import { ProductsService } from '../services/products.service';

@ApiTags('Products')
@ApiBearerAuth('access-token')
@Controller('api/v1/products')
export class ProductsController {
  constructor(
    private readonly createSkuUseCase: CreateSkuUseCase,
    @Inject(PRODUCTS_SERVICE) private readonly service: ProductsService,
  ) {}

  @Post('skus')
  @ApiOperation({ summary: 'Create a new SKU' })
  @ApiResponse({ status: 201, description: 'SKU created' })
  @ApiResponse({ status: 400, description: 'Validation error' })
  @ApiResponse({ status: 409, description: 'SKU name already exists' })
  async createSku(@Body() input: CreateSkuDto) {
    return await this.createSkuUseCase.execute(input);
  }
}
```

## Testing

### Unit test: CreateSkuUseCase

```typescript
// src/modules/products/use-cases/__tests__/create-sku.use-case.test.ts
import { CreateSkuUseCase } from '../create-sku.use-case';

describe('CreateSkuUseCase', () => {
  let useCase: CreateSkuUseCase;
  let driver: jest.Mocked<IProductsDriver>;

  beforeEach(() => {
    driver = {
      findSkuByName: jest.fn(),
      createSku: jest.fn(),
    } as any;
    useCase = new CreateSkuUseCase(driver);
  });

  it('creates a SKU with valid input', async () => {
    driver.findSkuByName.mockResolvedValue(null);
    driver.createSku.mockResolvedValue({ id: '123', name: 'Tomato' });

    const result = await useCase.execute({
      name: 'Tomato',
      category: 'PRODUCE',
      unit: 'KG',
      price: '500',
      storageType: 'AMBIENT',
      shelfLife: 7,
    });

    expect(result.id).toBe('123');
    expect(driver.createSku).toHaveBeenCalled();
  });

  it('throws conflict when SKU name exists', async () => {
    driver.findSkuByName.mockResolvedValue({ id: '1' });

    await expect(
      useCase.execute({
        name: 'Tomato',
        category: 'PRODUCE',
        unit: 'KG',
        price: '500',
        storageType: 'AMBIENT',
        shelfLife: 7,
      })
    ).rejects.toThrow('SKU name already exists');
  });
});
```

## Validation Rules

From the acceptance criteria:
- Name: required, 1–255 characters
- Category: required, must match enum
- Unit: required, enum value only
- Price: required, positive Decimal, no floating point
- Storage: required, enum value
- Shelf-life: required, integer >= 1

## Acceptance Checklist

- [ ] Module scaffolds without errors: `yarn build`
- [ ] Tests pass: `yarn test products`
- [ ] No TypeScript errors: `yarn lint:check`
- [ ] Swagger endpoint visible at `/docs`
- [ ] Endpoint tested: `curl -X POST http://localhost:3000/api/v1/products/skus -d '...'`
- [ ] Audit log entry created on successful POST
- [ ] Code follows flow-be/CLAUDE.md conventions
- [ ] PR passes CI/CD checks

## Definition of Done

1. Code is merged to main branch
2. Automated tests pass
3. Swagger docs are accurate
4. Code review approved by backend maintainer
5. Task marked as Done in Jira
6. Related story moves to "In Review" or "Done" status
