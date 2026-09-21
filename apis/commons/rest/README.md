# API Commons

Versioned OpenAPI components shared by the APIs in this workspace.

## Contents

- `v1/components/errors.yml`: shared RFC 7807 error schemas and HTTP responses.

Service-specific technical components such as pagination, sorting, and UUIDs remain in each API because their allowed fields and behavior belong to that API contract.

## Versioning

The current contract version is recorded in `VERSION` and is released as a Git tag using the format `api-commons-v<version>`, for example `api-commons-v1.0.0`.

Consumers must pin a tag or immutable commit instead of consuming the default branch. A release changes the version when the shared contract changes:

- Patch: clarifications or compatible examples/descriptions.
- Minor: new optional schemas or responses.
- Major: breaking changes to schemas, response names, or references.

## Consumption

Keep this directory as `apis/commons` in an API repository and reference the shared components with relative OpenAPI `$ref` values. When commons is maintained in a separate repository, vendor the directory at the pinned `api-commons-v<version>` tag or expose the same directory through the repository's contract dependency mechanism.

For example:

```yaml
$ref: '../../../../../commons/rest/v1/components/errors.yml#/components/responses/BadRequest400'
```
