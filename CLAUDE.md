# CLAUDE.md

## Project

Spring Boot 2.7 REST-to-SOAP adapter for the SoftExpert Forms web service. Exposes a JSON/XML REST API that translates requests into JAXB2-marshalled SOAP calls.

## Build & Run

```bash
# Run locally
./mvnw spring-boot:run

# Run tests
./mvnw verify

# Package JAR
./mvnw package
```

Swagger UI: `http://localhost:8080/swagger-ui/`

## Configuration

`application.yml` is gitignored. Create `src/main/resources/application.yml`:

```yaml
env:
  soft-expert:
    jwt: "Bearer <token>"
    domain: "https://<softexpert-host>"
    forms-endpoint: "/se/ws/fm_ws.php"
```

## WSDL Code Generation

The `maven-jaxb2-plugin` generates `src/main/java/com/softexpert/integration/wsdl/forms/` from the live SoftExpert WSDL on every build. Do not hand-edit files in that package — they will be overwritten. The WSDL source URL is configured in `pom.xml`.

## Key Packages

| Package | Role |
|---|---|
| `client` | `FormsClient` — Spring-WS gateway wrapping all SOAP operations |
| `config` | Bean wiring, property binding (`Settings`), SOAP header injection (`SoftExpertSoapRequestCallback`) |
| `controller` | REST endpoints: `/api/forms` (thin CRUD) and `/api/countries` (higher-level) |
| `service` | `CountriesService` — idempotent seeding of a SoftExpert countries table |
| `model` | Domain POJOs (ISO 3166 country/state data) |
| `wsdl/forms` | Auto-generated JAXB2 classes — do not edit |

## Auth

The JWT is injected as the `Authorization` MIME header on every SOAP request inside `SoftExpertSoapRequestCallback`. It is loaded from `Settings.JWT`, which is populated at startup from `env.soft-expert.jwt`.

## Tests

Tests live in `src/test/`. Run with `./mvnw test`. The project currently has a single smoke test (`IntegrationApplicationTests`).
