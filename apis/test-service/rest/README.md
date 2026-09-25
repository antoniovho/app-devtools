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
- Current version: `0.1.3`
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

## Project Scoping

Test Cases, Preconditions, Test Sets, Test Plans, and Executions belong to
exactly one project, so their collections are nested under
`/v1/projects/{projectKey}/...` (for example
`/v1/projects/{projectKey}/test-cases`). Business keys (`testKey`,
`preconditionKey`, `setKey`, `planKey`) are unique per project, not globally,
so every operation that resolves an entity by business key requires the owning
`projectKey` in the path.

Environments are not owned by a project and stay top-level at `/v1/environments`.
The `Projects` catalog itself (`/v1/projects`, `/v1/projects/{projectKey}`) is
also top-level, since it is the resource being scoped into.

## Versioned Resources

Test Cases, Preconditions, Test Sets, and Test Plans are immutable snapshots.
Their business keys (`testKey`, `preconditionKey`, `setKey`, and `planKey`)
identify logical entities within their owning project. Each response `id`
identifies one immutable UUID snapshot, while `version` is its positive
monotonic version number.

Lifecycle is independent of versioning. The supported snapshot statuses are
`DRAFT`, `ACTIVE`, and `DEPRECATED`. Content changes require a new snapshot;
the contract exposes version, activation, and deprecation operations rather than
`PATCH` operations.

Each versioned resource exposes two distinct ways to reach its versions,
which are not interchangeable:

- `POST /v1/projects/{projectKey}/test-cases/{testCaseId}/versions` creates a
  new version whose parent is the exact snapshot `{testCaseId}` (the `source`
  version being branched from). The equivalent exists for preconditions, test
  sets, and test plans.
- `GET /v1/projects/{projectKey}/test-case-keys/{testKey}/versions` lists every
  version sharing the logical `testKey`, regardless of which version is the
  parent of which. Supports `status` (`DRAFT`/`ACTIVE`/`DEPRECATED`),
  `sortBy`, and `order`. The equivalent exists for precondition, test set, and
  test plan keys (`precondition-keys`, `test-set-keys`, `test-plan-keys`).

Test Sets and Test Plans reference exact version UUIDs. An execution therefore
uses the stored plan snapshot rather than resolving a later version.

## Execution and Viewer Rules

`POST /v1/projects/{projectKey}/executions` schedules asynchronous work and
returns `202 Accepted`. Execution history is identified by `Execution.id` and
is not versioned.

`ActionResult` records the outcome of one `Definition.actions[].id`. Large
payloads, logs, streams, and binary evidence belong in `TestResultArtifact`,
not in `expected`, `actual`, or `output`.

PostgreSQL is the intended system of record. Viewer Integration exposes
projection and drift resources only: an external viewer, including Xray, must
not update Test Service domain state.

`POST /v1/projects/{projectKey}/viewer/publications` and
`POST /v1/projects/{projectKey}/viewer/drift-checks` are batch operations: they
publish, or check drift for, the current `ACTIVE` version of every Test Case,
Precondition, Test Set, and Test Plan in the project. `ViewerSyncRecord` and
`DriftEvent` both carry a `project` reference, since entity keys are only
unique per project. Synchronization records and drift events can be listed
either per project (`/v1/projects/{projectKey}/viewer/sync-records`,
`/v1/projects/{projectKey}/viewer/drift-events`) or across every project
(`/v1/viewer/sync-records`, `/v1/viewer/drift-events`) for cross-project
auditing.

## Validate Locally

Run from the repository root:

```bash
npx --yes @redocly/cli@1.34.20 lint apis/test-service/rest/openapi-rest.yml

npx --yes @redocly/cli@1.34.20 bundle \
  apis/test-service/rest/openapi-rest.yml \
  --output /tmp/test-service-api-0.1.3.yml

openapi-generator-cli validate -i /tmp/test-service-api-0.1.3.yml

```

The bundle is a single OpenAPI YAML document with resolved references. It is not
generated server code or a client library.

## Release and Consumption

The release workflow resolves the API through `apis/metadata.yml`, validates
that `metadata.yml` and `openapi-rest.yml` declare the same version, bundles the
contract, and publishes an immutable GitHub Release from `main`.

For version `0.1.3`:

```text
Tag:      test-service-api-v0.1.3
Asset:    test-service-api-0.1.3.yml
```

Consumers must download and pin a specific release asset. Do not generate from,
or otherwise use, the default branch as a production contract dependency.

Generated code is disposable. Keep business rules and persistence outside the
generated directory, and map generated HTTP types to application ports in the
consumer backend.
