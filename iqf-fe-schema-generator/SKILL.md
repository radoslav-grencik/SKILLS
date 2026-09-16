---
name: iqf-fe-schema-generator
description: >-
  Generate, find, and update frontend Zod schemas for IQF backend-to-frontend contract mapping. Use when an IQF task mentions Zod schemas, FE schemas, schema.ts, schemas/*.ts, Zod exports, backend @View interfaces/classes, DTOs, Java records, commands, BrowseSchema, DetailSchema, FormSchema, SaveSchema, UpdateSchema, module schema sync, or aligning frontend schemas with current BE sources. Also use after editing an IQF backend @View, DTO, Java record, command, payload, or similar API contract when the serialized contract shape changed or likely affects FE validation/types. Do not trigger for unrelated backend/frontend cleanup just because the repository is IQF.
---

# IQF FE Schema Generator

Generate, find, and update frontend Zod schemas for IQF modules from backend `@View` classes, `Dto` classes, Java records, and similar payload classes. Keep the generated FE schemas aligned with the backend projection model and with existing FE module conventions in the current project.

## First Questions

Before generating new schemas, ask for the backend module unless the user already answered it clearly:

1. For which IQF BE module should schemas be generated? Ask for a module/path/name such as a Gradle/Maven module, package path, bounded-context name, or domain name.

Do not ask whether to use one `schema.ts` file or split `schemas/*.ts` files when the target FE module already has a clear local convention. Discover and follow the existing layout. Ask the layout question only when generating schemas into a new or ambiguous FE module with no clear precedent.

If the user asks to find or update existing schemas, search for the existing schema files and update them in place. Ask only if multiple plausible FE modules or schema sets match.

## Project Discovery

At the start of work in any repository, discover the local IQF layout instead of assuming project-specific paths:

- Locate backend modules by searching for Java sources containing `@View(`, `BaseObject`, `DomainObject`, `TitledObject`, `DictionaryObject`, `Dto` classes, Java records, and payload/command classes.
- Locate frontend modules by searching for `package.json`, `tsconfig.json`, `vite.config.*`, `next.config.*`, `src/modules`, `schema.ts`, and `schemas/*.ts`.
- Locate existing IQF FE base schemas by searching for exports/imports of `baseSchema`, `domainSchema`, `titledSchema`, and `dictionarySchema`.
- Locate package aliases from `tsconfig.json`, bundler config, and existing imports before adding new imports.
- Inspect nearby schema files to learn whether the project prefers one `schema.ts`, split `schemas/*.ts`, barrel exports, lower-camel schema names, PascalCase schema names, or another convention.

Do not hard-code module roots like `apps/*`, `packages/*`, or a specific project prefix unless those roots are present in the current repository.

Existing FE schemas are useful for layout, naming, imports, Zod style, and existing FE-only validation structure. They are not the source of truth for fields, field types, nullability, requiredness, or validation because the backend may have changed. Always re-check the current BE source before preserving, adding, removing, or changing fields, nullability, and validation rules.

## Repository Workflow

1. Locate the backend module and inspect relevant Java files.
2. Find classes annotated with `@View(...)`; these drive read schemas.
3. Find relevant `Dto`, Java record, command, or payload classes; these usually drive form/edit validation schemas.
4. Locate the target frontend module using the discovered FE layout and existing module naming.
5. Inspect existing nearby schemas, exports, API callers, hooks, and naming conventions before editing.
6. Generate or update only the minimal set of schema files needed.
7. Update exports (`index.ts`) only when the local module pattern exports schemas from there.
8. Run the narrowest practical verification available, such as TypeScript/package check or a targeted grep/read review when a full check is too expensive.

## Automatic Sync After Backend Contract Edits

When you create, edit, rename, or remove an IQF backend `@View`, `Dto`, Java `record`, command, payload, or similar API contract class during any task, treat the FE Zod schema sync as part of the same task before finishing if the serialized contract shape changed or likely affects FE validation/types. This applies even when the original user request only mentioned backend work.

After such a backend contract edit:

1. Identify which frontend schemas, inferred types, API callers, forms, tables, or hooks consume the changed backend contract.
2. Re-read the final backend source after your BE edits; do not rely on the pre-edit shape or memory of the change.
3. Update the affected FE Zod schemas in place using the normal mapping, nullability, validation, inheritance, and cross-module rules in this skill.
4. Remove FE fields for removed backend contract members, add FE fields for new members, and update types/nullability/validation for changed members.
5. If no matching FE schema exists, search for the relevant FE module and create the schema only when the local layout and scope are clear. If the target is ambiguous or the change would expand into unrelated modules, report the ambiguity before making broad edits.
6. Run the narrowest practical verification for both the backend edit and the affected frontend schema code.

Do not skip this sync just because the user did not explicitly ask for Zod changes. In IQF projects, backend projection/payload contracts and FE schemas are coupled; leaving them out of sync creates runtime validation and type safety failures.

## Backend Sources

Focus on these backend inputs:

- `@View(...)` classes/interfaces: generate read schemas from view fields and JavaBean getter methods. IQF views are often interfaces, so read method return types, annotations, generic collection item types, inherited interface methods, and implemented/removed view fragments; do not search only for fields.
- `*BrowseView`, `*ListView`, `*LabeledView`, `*IdView`: usually map to browse/list/id/reference schemas used by tables, options, and nested relations.
- `*DetailView`: usually maps to detail schemas used by data forms and detail pages.
- `*CreateView`, `*UpdateView`: may map to create/update/detail schemas depending on the endpoint contract.
- `*Dto`, `*SaveDto`, `*UpdateDto`, `*CreateDto`, Java records, commands, and similar payload classes: usually map to form schemas used for edit/create validation.
- Superclasses, parent interfaces, and inherited fields/getters; never ignore inherited members when they are part of the serialized contract.

When a view extends another view class, read the parent view too and represent the inherited shape in FE by extending or composing the corresponding FE schema if one exists.

## Base Object Mapping

Backend entities typically inherit through this hierarchy:

- `BaseObject`
- `DomainObject`
- `TitledObject`
- `DictionaryObject`

Frontend equivalents are usually existing IQF FE schemas. In many IQF projects they are imported from `iqf-web-ui`, but always confirm the actual import paths in the current repository:

- `BaseObject` -> `baseSchema`
- `DomainObject` -> `domainSchema`
- `TitledObject` -> `titledSchema`
- `DictionaryObject` -> `dictionarySchema`

For read schemas, choose the most specific existing base schema that matches the BE class hierarchy. Do not duplicate base fields in every generated schema when an FE base schema already provides them.

For `FormSchema`, `SaveSchema`, and `UpdateSchema`, first identify the backend source of the editable contract:

- If the source is a BE `@View` class/interface, the generated form/update schema should extend `baseSchema`. IQF `@View` payloads expose object identity through the view contract, so keep that base object shape even when the corresponding read/detail schema extends `domainSchema`, `titledSchema`, or `dictionarySchema`.
- If the source is a `Dto`, `SaveDto`, `UpdateDto`, `CreateDto`, Java `record`, command object, or similar payload class, generate a plain `z.object(...)` that contains only the editable payload fields. Do not add `baseSchema` just because the related entity or read/detail view inherits from `BaseObject`.
- Use a more specific base schema for form/update schemas only when the actual BE form/update source is a `@View` contract that exposes it, nearby form/update schemas already establish that convention for the same source type, or the backend payload contract clearly requires those inherited base fields.

## Schema Types

Typical FE schemas are:

- `BrowseSchema`: for data table/list responses; based on `*BrowseView`, `*ListView`, `*LabeledView`, or equivalent read views.
- `DetailSchema`: for detail pages and data forms; based on `*DetailView` and related detail views.
- `FormSchema`/`SaveSchema`/`UpdateSchema`: for data form validation in edit/create mode; usually based on `Dto`, `SaveDto`, `UpdateDto`, `CreateDto`, Java `record`, command object, or a form/update `@View`. Extend `baseSchema` when the source is a BE `@View`; use plain `z.object(...)` when the source is a DTO/record/payload class.

Match existing naming style in the module. Many IQF frontends use lower-camel exports such as `monitoringBrowseSchema`, `monitoringDetailSchema`, `monitoringUpdateSchema`, and types like `MonitoringBrowse`, but some projects may use PascalCase schema names. Follow the local convention.

If the user explicitly says `FormSchema`, prefer `...FormSchema` naming only if nearby modules use that naming. Otherwise preserve local conventions such as `...SaveSchema` or `...UpdateSchema`.

## File Layout Decision

When the user chooses one file:

- Put schemas and inferred types in the target FE module's top-level `schema.ts`.
- Preserve any existing imports, export ordering, and local formatting style.

When the user chooses split files:

- Put files under the target FE module's top-level `schemas/` directory.
- Use existing filename style, often kebab-case like `monitoring-browse.ts`, `monitoring-detail.ts`, `monitoring-update.ts`.
- Add or update `schemas/index.ts` only when the surrounding package/module already uses barrel exports.

If the target FE module already has a clear layout, follow it unless the user explicitly asks to migrate layout.

## Cross-Module Views

Backend views often reference views from other modules. Always resolve those references end-to-end:

1. Read the referenced BE view or DTO class.
2. Search the FE codebase for a matching schema.
3. If a matching FE schema exists, import and reuse it.
4. If it does not exist, create the missing schema only when it is necessary for the requested output and the target FE module/layout is clear. Otherwise report the missing referenced schema and ask before expanding the edit scope across modules.
5. Prefer existing public import paths and package aliases when nearby code uses them; otherwise use relative imports consistent with the file being edited.

Do not replace a specific referenced view with `baseSchema` unless the BE field only exposes base fields or there is no practical view contract to model. Specific nested schemas are valuable because they keep data tables and forms type-safe.

## Type Mapping

Use the repository's existing Zod style:

- Java `String`, `UUID`, identifiers -> `z.string()` unless existing schemas use a stricter helper.
- Java date/time types -> usually `z.string()` in FE unless nearby code uses another representation.
- Java numeric types -> `z.number()`.
- Java boolean types -> `z.boolean()`.
- Java enum/value objects represented through IQF enumerated helpers -> use existing `createEnumeratedSchema(...)` and value schemas when present.
- Java collections -> `z.array(itemSchema)`.
- Java maps/unknown dynamic JSON -> inspect existing conventions before choosing `z.record(...)` or `z.unknown()`.
- Nullable/optional backend fields -> determine from the current BE source of truth; render as `.nullish()` on the FE unless the local project has a clearly different established convention.

When converting DTOs to form schemas, distinguish API read nullability from form validation. If a DTO/form field is not proven required by validation annotations or discovered validation logic, keep it permissive with `.nullish()` on the FE. Add stricter form constraints only when the backend contract or validation logic proves them.

When updating an existing form/update schema, treat the backend as the source of truth for field presence, field types, nullability, requiredness, and validation. Preserve existing FE-only `.refine(...)`, `.superRefine(...)`, transforms, helper functions, and localized validation messages only when they still match the current backend/form contract. Update or remove refinement rules when backend fields disappear, new fields are added, types change, nullability changes, validation annotations change, or BE validators/services enforce different behavior. Existing FE refinements may encode cross-field rules that are not visible in DTO annotations, but they must not contradict the current backend.

## Nullability And Validation

Read schemas (`BrowseSchema`, `DetailSchema`, and other API data schemas) model API data shape. Form schemas model edit/create validation. Do not copy form-only validation constraints into read schemas.

For read schemas:

- Treat fields as nullable/optional according to the current backend view contract, not according to the previous FE schema.
- Add newly exposed backend view fields and remove fields that no longer exist in the current backend view contract.
- Update field schemas when backend field types, nested view types, collection item types, or enum/value representations change.
- Use `.nullish()` when the backend field can be absent or null.
- Keep fields required only when the backend view contract clearly guarantees presence.

For form schemas from DTOs, records, commands, and similar payload classes:

- Consider a field required only when the property has an explicit non-null validation annotation or equivalent validation logic. Common annotations include `@NotNull`, `@NonNull`, `@Nonnull`, `@NotBlank`, `@NotEmpty`, and project-specific aliases/wrappers around these annotations. Confirm imports because annotation names can come from different packages.
- Inspect validation annotations on fields, getters, constructor parameters, and relevant superclass fields.
- Inspect usages of the DTO, generated `@View` class, controller/service methods, validators, mappers, and form submission code for additional validation logic that affects requiredness or constraints.
- Add non-null and other validation rules only to `FormSchema`/save/update schemas, not to browse/detail/read schemas.
- Do not infer required form fields only from Java primitive types, database column metadata, field names, or domain intuition. Require an annotation or discovered validation path.
- Add newly exposed payload fields and remove fields that no longer exist in the current payload/form contract.
- Update field schemas when DTO/record/payload field types, nested DTO/view types, collection item types, enum/value representations, nullability, or validation annotations change.
- Use existing FE schemas only to preserve local Zod expression style, helpers, and message conventions. Do not preserve old fields, nullability, or validation constraints unless they are still backed by the current BE source or discovered validation logic.
- Preserve existing FE-only cross-field validation (`.refine(...)`, `.superRefine(...)`), helper functions, transforms, and messages when updating form/update schemas only if they still match the current BE/form contract. Update or remove them when BE fields, nullability, validation annotations, validators, services, or form submission behavior changed.

Map Java validation annotations to Zod only when they are present or enforced by discovered validation logic:

- `@NotNull`, `@NonNull`, `@Nonnull` -> required field; do not add `.nullish()`.
- `@NotBlank` -> required string with the local non-empty/trim validation style.
- `@NotEmpty` -> required non-empty string/array/collection according to field type.
- `@Size(min = ..., max = ...)` -> string/array length constraints when supported by local style.
- `@Min`, `@Max`, `@Positive`, `@PositiveOrZero`, `@Negative`, `@NegativeOrZero` -> numeric constraints.
- `@Pattern` -> regex validation only if existing code already uses comparable regex validation or the pattern is simple and safe to copy.
- `@Email` -> email validation if local form schemas use Zod email validation.

If validation is unclear, prefer the less restrictive schema and mention the ambiguity in the final response instead of silently inventing constraints.

## Update Requests

When the user asks to find or update schemas:

- Search for schema exports, filenames, inferred types, API usage, and imports.
- Read the BE source that owns the contract before changing FE schemas.
- Treat BE as the source of truth: add fields that were introduced, remove fields that disappeared, and update field types, nested schemas, nullability, requiredness, and validation when the BE contract changed.
- Preserve user changes and unrelated edits.
- Update dependent schemas if the BE change affects nested view fields.
- Keep changes small; do not rewrite module structure unless necessary.

## Quality Checklist

Before finishing:

- Every generated read schema has a clear BE `@View` source.
- Every generated form schema has a clear BE source: DTO/record/payload class or form/update `@View`.
- Read schemas extend the correct FE base schema for the BE inheritance chain.
- Form/update schemas extend `baseSchema` only for BE `@View` sources or when the actual payload contract requires base fields; DTO/record/payload sources use plain `z.object(...)`.
- Existing FE-only refinements, transforms, helper functions, and localized validation messages are updated or removed when they no longer match the current BE/form contract.
- Cross-module BE views are resolved to existing schemas, newly created schemas within the agreed scope, or explicitly reported as missing.
- Imports follow local alias/relative style.
- Exported schema and type names match local casing conventions.
- Existing API calls/forms that need the schema are updated only when required.
- TypeScript/Zod syntax is valid.
- The final response mentions unresolved contract ambiguities, missing referenced schemas that were not created, and the verification that was run or why it was skipped.

## Useful Searches

Use these search patterns as starting points:

```text
@View\(
class .*Dto|record .*Dto|record .*Command|record .*Payload|class .*Command|class .*Payload
@NotNull|@NonNull|@Nonnull|@NotBlank|@NotEmpty|@Size|@Min|@Max|@Pattern|@Email
new .*Dto|\.toDto\(|DtoMapper|Validator|validate\(
extends BaseObject|extends DomainObject|extends TitledObject|extends DictionaryObject
BrowseSchema|DetailSchema|FormSchema
export const .*BrowseSchema|export const .*DetailSchema|export const .*FormSchema
baseSchema|domainSchema|titledSchema|dictionarySchema
```

Adapt casing because IQF projects differ: some use lower-camel exports such as `monitoringBrowseSchema`, while others may use PascalCase names.
