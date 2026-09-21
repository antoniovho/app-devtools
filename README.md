# app-devtools

Repository containing versioned OpenAPI contracts for DevTools APIs.

## APIs

Metadata:

- `apis/metadta.yml` funciona como catálogo/discovery del pipeline
metadata propia de esa API.
- `apis/<api>/rest/metadata.yml` es metadata propia de esa API. Es el **source of truth** del versionado de distribución.

```text
apis/test-service/rest/metadata.yml
        │
        │ api.version
        ▼
     0.1.0
        │
        ├── OpenAPI info.version debe coincidir
        ├── tag = test-service-api-v0.1.0
        ├── release = test-service-api-v0.1.0
        └── bundle = test-service-api-0.1.0.yml
```

Para validación de PR, sí:

`metadata.version >= última versión publicada`

Porque durante desarrollo puedes seguir trabajando sobre 0.2.0 aunque todavía no esté publicada.

Pero para crear una nueva release, debe ser estrictamente:

`metadata.version > última versión publicada`

```text
Última release: 0.1.0

metadata 0.1.0
    ├── validate → OK
    └── release  → ERROR: ya existe

metadata 0.1.1
    ├── validate → OK
    └── release  → OK

metadata 0.2.0
    ├── validate → OK
    └── release  → OK

metadata 0.0.9
    ├── validate → ERROR
    └── release  → ERROR
```

### Test Service API

Source:

`apis/test-service/rest/openapi-rest.yml`

The source contract may reference components and services from multiple YAML files.

This repository contains OpenAPI contracts only. Backend implementation and
generated code belong in the Test Service backend repository. Consumers must use
an immutable GitHub Release asset rather than the default branch.

## Validation

OpenAPI contracts are validated automatically by GitHub Actions on pull requests.

Local validation:

```bash
npx --yes @redocly/cli@1.34.3 lint test-service
npx --yes @redocly/cli@1.34.3 bundle test-service \
	--output /tmp/test-service-api.yml \
	--component-renaming-conflicts-severity=error
