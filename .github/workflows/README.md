```text
                    apis/metadata.yml
                           │
                 id: test-service
                           │
             definition-path: test-service/rest
                           │
                           ▼
          apis/test-service/rest/metadata.yml
                           │
                 version: 0.1.0
                 definition:
                   openapi-rest.yml
                           │
                           ▼
          apis/test-service/rest/openapi-rest.yml
                           │
                 info.version: 0.1.0
                           │
                           ▼
                    Redocly lint
                           │
                    Redocly bundle
                           │
                           ▼
              test-service-api-0.1.0.yml
                           │
                           ▼
                 Git tag + Release
              test-service-api-v0.1.0
```
