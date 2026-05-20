# SoftExpert SOAP Integration

A Spring Boot REST API that acts as an adapter layer over the [SoftExpert](https://www.softexpert.com/) Forms SOAP web service. REST clients send JSON/XML to this service, which translates requests into SOAP calls, forwards them to SoftExpert, and returns the responses.

## Overview

SoftExpert exposes its Forms module as a WSDL/SOAP service. This project wraps that interface in a conventional REST API with Swagger documentation, making it easier to consume from modern clients.

WSDL Java classes are auto-generated at build time from the SoftExpert WSDL endpoint via the `maven-jaxb2-plugin`.

## Tech Stack

| Technology | Version |
|---|---|
| Java | 8 |
| Spring Boot | 2.7.10 |
| Spring Web Services (spring-ws) | managed by Boot |
| JAXB2 (maven-jaxb2-plugin) | 0.14.0 |
| Springfox Swagger | 3.0.0 |

## Configuration

Configuration is loaded from `application.yml` (excluded from version control). Create `src/main/resources/application.yml` with the following:

```yaml
env:
  soft-expert:
    jwt: "Bearer <your-jwt-token>"
    domain: "https://<your-softexpert-domain>"
    forms-endpoint: "/se/ws/fm_ws.php"
```

| Property | Description |
|---|---|
| `env.soft-expert.jwt` | JWT bearer token for SoftExpert authentication, injected as the `Authorization` SOAP header |
| `env.soft-expert.domain` | Base URL of the SoftExpert instance |
| `env.soft-expert.forms-endpoint` | Path to the SoftExpert Forms SOAP endpoint |

## Running

```bash
./mvnw spring-boot:run
```

Swagger UI is available at `http://localhost:8080/swagger-ui/` once the service is running.

## API Endpoints

### Forms — `/api/forms`

Thin CRUD wrapper over the SoftExpert `fm_ws` SOAP operations. All endpoints accept and produce both `application/json` and `application/xml`.

| Method | Path | SOAP Action | Description |
|---|---|---|---|
| `POST` | `/api/forms` | `urn:form#getTableRecord` | Fetch records from a form table |
| `POST` | `/api/forms/new` | `urn:form#newTableRecord` | Create a single form table record |
| `PUT` | `/api/forms` | `urn:form#editTableRecord` | Edit a form table record |
| `DELETE` | `/api/forms` | `urn:form#deleteTableRecord` | Delete a single form table record |
| `POST` | `/api/forms/list` | `urn:form#newTableRecordList` | Bulk-create form table records |
| `DELETE` | `/api/forms/list` | `urn:form#deleteTableRecordList` | Bulk-delete form table records |

### Countries — `/api/countries`

Higher-level service that manages a `countries` table in SoftExpert. Accepts ISO 3166 country data and only inserts records that do not already exist (deduplicates on `alpha-2` code).

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/countries` | Idempotently create countries in SoftExpert |

**Request body** accepts fields compatible with the [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html) country list format:

```json
{
  "countries": [
    {
      "name": "United States of America",
      "alpha-2": "US",
      "alpha-3": "USA",
      "country-code": "840",
      "region": "Americas",
      "sub-region": "Northern America"
    }
  ]
}
```

## Project Structure

```
src/main/java/com/softexpert/integration/
├── IntegrationApplication.java       # Spring Boot entry point
├── client/
│   └── FormsClient.java              # Spring-WS gateway; executes SOAP operations
├── config/
│   ├── FormsClientConfig.java        # JAXB2 marshaller + FormsClient bean wiring
│   ├── Settings.java                 # Binds application.yml properties to static fields
│   ├── SoftExpertSoapRequestCallback.java  # Injects SOAPAction + Authorization headers
│   └── SpringFoxConfig.java          # Swagger/Springfox configuration
├── controller/
│   ├── FormsController.java          # REST endpoints for /api/forms
│   └── CountriesController.java      # REST endpoint for /api/countries
├── model/
│   ├── Country.java                  # ISO 3166 country model
│   ├── Countries.java                # Wrapper list
│   ├── StateOrProvince.java
│   └── StatesOrProvinces.java
├── service/
│   └── CountriesService.java         # Business logic for idempotent country seeding
├── util/
│   └── PrettyLogger.java             # Debug-level JSON pretty-printer for SOAP responses
└── wsdl/forms/                       # Auto-generated JAXB2 classes from SoftExpert WSDL
```

## Building

```bash
# Compile and run tests
./mvnw verify

# Build executable JAR
./mvnw package
```

> The `maven-jaxb2-plugin` fetches and parses the SoftExpert WSDL on every build to regenerate the `wsdl/forms` package. Ensure the SoftExpert instance is reachable during builds, or cache the generated sources.
