# Test Service API

Test Service is an experimental OpenAPI 3.0.3 REST contract for authoring
immutable test definitions, composing reproducible test plans, scheduling their
execution, and publishing read-only projections to an external testing viewer.

This directory contains contract source only. Backend implementation, generated
server stubs, generated clients, Python tooling, and virtual environments belong
to consumer repositories.

## Contract Source

- Root document: `openapi-rest.yml`
- API metadata: `metadata.yml`
- Current version: `0.1.0`
- Base path: `/v1`
- Local server: `http://localhost:8080`
- Authentication: Bearer JWT (`bearerAuth`)

The root document references service and component files under `v1/`. Bundle the
contract before distributing it or passing it to code generators.

## API Scope

| Area               | Resources                                                                      |
| ------------------ | ------------------------------------------------------------------------------ |
| Projects           | Project catalog and administrative lifecycle.                                  |
| Authoring          | Versioned Test Cases and Preconditions.                                        |
| Composition        | Versioned Test Sets and Test Plans.                                            |
| Execution          | Environments, asynchronous executions, results, action results, and artifacts. |
| Viewer Integration | Outbound publications, synchronization records, and drift events.              |

All operations are secured by the global Bearer JWT requirement. Error responses
use `application/problem+json` and the shared RFC 9457 Problem Details schema
from `apis/commons/rest/v1/components/errors.yml`.

## Versioned Resources

Test Cases, Preconditions, Test Sets, and Test Plans are immutable snapshots.
Their business keys (`testKey`, `preconditionKey`, `setKey`, and `planKey`)
identify logical entities. Each response `id` identifies one immutable UUID
snapshot, while `version` is its positive monotonic version number.

Lifecycle is independent of versioning. The supported snapshot statuses are
`DRAFT`, `ACTIVE`, and `DEPRECATED`. Content changes require a new snapshot;
the contract exposes version, activation, and deprecation operations rather than
`PATCH` operations.

Test Sets and Test Plans reference exact version UUIDs. An execution therefore
uses the stored plan snapshot rather than resolving a later version.

## Execution and Viewer Rules

`POST /v1/executions` schedules asynchronous work and returns `202 Accepted`.
Execution history is identified by `Execution.id` and is not versioned.

`ActionResult` records the outcome of one `Definition.actions[].id`. Large
payloads, logs, streams, and binary evidence belong in `TestResultArtifact`,
not in `expected`, `actual`, or `output`.

PostgreSQL is the intended system of record. Viewer Integration exposes
projection and drift resources only: an external viewer, including Xray, must
not update Test Service domain state.

## Validate Locally

Run from the repository root:

```bash
npx --yes @redocly/cli@1.34.20 lint apis/test-service/rest/openapi-rest.yml

npx --yes @redocly/cli@1.34.20 bundle \
  apis/test-service/rest/openapi-rest.yml \
  --output /tmp/test-service-api-0.1.0.yml

openapi-generator-cli validate -i /tmp/test-service-api-0.1.0.yml

```

The bundle is a single OpenAPI YAML document with resolved references. It is not
generated server code or a client library.

## Release and Consumption

The release workflow resolves the API through `apis/metadata.yml`, validates
that `metadata.yml` and `openapi-rest.yml` declare the same version, bundles the
contract, and publishes an immutable GitHub Release from `main`.

For version `0.1.0`:

```text
Tag:      test-service-api-v0.1.0
Asset:    test-service-api-0.1.0.yml
```

Consumers must download and pin a specific release asset. Do not generate from,
or otherwise use, the default branch as a production contract dependency.

Generated code is disposable. Keep business rules and persistence outside the
generated directory, and map generated HTTP types to application ports in the
consumer backend.
