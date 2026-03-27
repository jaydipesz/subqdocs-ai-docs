# Skill: Add a Backend Module

**When NOT to use this:** Adding a field to an existing model (migration + model edit only), adding a new route to an existing module (skip steps 1–5), or adding a cron job (use skill 07 instead).

---

## Files (in creation order)
```
subqdocs-backend/src/
  sequelize/migrations/<timestamp>-<name>.js
  sequelize/models/<name>.model.ts
  sequelize/models/types/<name>.model.type.ts
  sequelize/models/index.ts                    ← edit: register model
  sequelize/repository/<name>.repository.ts
  modules/<name>/validation_schema/<name>.validation.ts
  modules/<name>/controller/<name>.controller.ts
  modules/<name>/routes/<name>.route.ts
  server.ts                                    ← edit: register route + cron
```

---

## Step 1 — Migration

```bash
cd subqdocs-backend
npx sequelize-cli migration:create --name add-my-entity
# edit the generated file in src/sequelize/migrations/
npm run migrate
```

---

## Step 2 — Model

```ts
// src/sequelize/models/my_entity.model.ts
import {
    AllowNull, AutoIncrement, BelongsTo, Column, CreatedAt,
    DataType, DeletedAt, ForeignKey, Model, PrimaryKey, Table, UpdatedAt,
} from 'sequelize-typescript';
import Organization from '@models/organization.model';
import { MyEntityAttributesType, RequiredMyEntityAttributesType } from '@models/types/my_entity.model.type';

@Table({ timestamps: true, paranoid: true, tableName: 'my_entities' })
export default class MyEntity extends Model<MyEntityAttributesType, RequiredMyEntityAttributesType> {
    @PrimaryKey @AutoIncrement @AllowNull(false) @Column(DataType.INTEGER) id: number;

    @AllowNull(false) @Column(DataType.STRING(255)) name: string;

    @ForeignKey(() => Organization) @AllowNull(false) @Column(DataType.INTEGER) organization_id: number;

    @CreatedAt created_at: Date;
    @UpdatedAt updated_at: Date;
    @DeletedAt deleted_at: Date;   // paranoid: true requires this

    @BelongsTo(() => Organization, { foreignKey: 'organization_id' }) organization: Organization;
}
```

**`paranoid: true` is mandatory for any model that holds patient/clinical data.** Without it, `.destroy()` hard-deletes.

---

## Step 3 — Type interface

```ts
// src/sequelize/models/types/my_entity.model.type.ts
export interface MyEntityAttributesType {
    id: number;
    name: string;
    organization_id: number;
    created_at?: Date;
    updated_at?: Date;
    deleted_at?: Date;
}
export type RequiredMyEntityAttributesType = Pick<MyEntityAttributesType, 'name' | 'organization_id'>;
```

---

## Step 4 — Register model

In `src/sequelize/models/index.ts`, add `MyEntity` to the `models` array passed to `new Sequelize({ models: [...] })`. Without this the model is invisible to Sequelize — no error is thrown, queries silently fail.

---

## Step 5 — Repository

```ts
// src/sequelize/repository/my_entity.repository.ts
import MyEntity from '@models/my_entity.model';
import { getRepository } from '.';

const MyEntityRepo = getRepository<MyEntity>(MyEntity.name);

export const getMyEntityRepo    = MyEntityRepo.get;
export const getMyEntitiesRepo  = MyEntityRepo.getAll;
export const createMyEntityRepo = MyEntityRepo.create;
export const updateMyEntityRepo = MyEntityRepo.update;
export const deleteMyEntityRepo = MyEntityRepo.deleteData;  // soft-delete (paranoid)
```

---

## Step 6 — Validation schema

```ts
// src/modules/my-module/validation_schema/my-module.validation.ts
import Joi from 'joi';

export const createMyEntitySchema = Joi.object({
    name: Joi.string().max(255).required(),
});
```

> Joi is the primary validation library. Some modules use `express-validator` or `zod` — match whatever the adjacent modules use.

---

## Step 7 — Controller

```ts
// src/modules/my-module/controller/my-module.controller.ts
import { Request, Response } from 'express';
import { parse } from '@utils/common.utils';
import generalResponse from '@utils/generalResponse';
import { logger } from '@utils/logger';
import { createMyEntityRepo } from '@repositories/my_entity.repository';

export const createMyEntity = async (req: Request, res: Response) => {
    try {
        const loggedInUser = req.user;
        const { name } = req.body;

        const entity = await createMyEntityRepo({
            name,
            organization_id: loggedInUser.organization_id,  // ← MANDATORY on every write
        });
        return generalResponse(res, parse(entity), 'Created successfully', 'success', true, 200);
    } catch (error) {
        logger.error('createMyEntity failed', error);
        return generalResponse(res, null, error.message || 'An error occurred', 'error', true, 500);
    }
};
```

`parse()` turns a raw Sequelize model instance into a plain JS object. Skipping it causes JSON serialization issues.

---

## Step 8 — Route (factory function — mandatory)

```ts
// src/modules/my-module/routes/my-module.route.ts
import { Router } from 'express';
import { authMiddleware, organizationMemberMiddleware } from '@middlewares/auth.middleware';
import { trackDeviceLog } from '@middlewares/trackDeviceLog.middleware';
import { userActivityMiddleware } from '@middlewares/userActivity.middleware';
import validationMiddleware from '@middlewares/middleware';
import { createMyEntitySchema } from '../validation_schema/my-module.validation';
import { createMyEntity } from '../controller/my-module.controller';

const MyModuleRoute = (): Router => {
    const router = Router();

    router.post(
        '/my-entity',
        authMiddleware,                                         // JWT + session check
        organizationMemberMiddleware,                          // patient_id/visit_id org scope (if body has them)
        trackDeviceLog,
        userActivityMiddleware,
        validationMiddleware(createMyEntitySchema, 'body'),
        createMyEntity,
    );

    return router;
};
export default MyModuleRoute;
```

A plain `export default Router()` does **not** work with `server.ts` — the array expects invoked factory functions: `MyModuleRoute()`.

---

## Step 9 — Register in server.ts

```ts
// src/server.ts — add import
import MyModuleRoute from '@modules/my-module/routes/my-module.route';

// inside main():
const apiRoutes = [
    // ... existing ...
    MyModuleRoute(),
];
```

---

## Exact codebase reference

From `src/modules/patient/routes/patient.route.ts`:
```ts
const PatientRoute = (): Router => {
    const PatientRouter: Router = Router();
    const BASE_PATH = '/patient';

    PatientRouter.post(
        `${BASE_PATH}/create`,
        authMiddleware,
        trackDeviceLog,
        userActivityMiddleware,
        validationMiddleware(createPatientSchema, 'body'),
        createPatient,
    );

    return PatientRouter;
};
export default PatientRoute;
```

---

## Checklist
- [ ] Migration generated and run
- [ ] `@Table({ paranoid: true })` on clinical/patient models
- [ ] Model registered in `src/sequelize/models/index.ts`
- [ ] Every write includes `organization_id: loggedInUser.organization_id`
- [ ] Controller calls `parse(model)` before responding
- [ ] Route is a factory function (`const X = (): Router => { ... }`)
- [ ] Route added to `apiRoutes` array in `server.ts`
