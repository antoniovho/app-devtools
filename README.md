# app-devtools

Repository containing versioned OpenAPI contracts for DevTools APIs.

## APIs

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
