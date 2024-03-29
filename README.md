# STAC API - Language (I18N) Extension

- **Title:** Language (I18N)
- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance Classes:**
  - <https://api.stacspec.org/v1.0.0-rc.2/language>
- **Scope:** STAC API - Core
- **[Extension Maturity Classification](https://github.com/radiantearth/stac-api-spec/tree/main/README.md#maturity-classification):** Pilot
- **Dependencies:**
  - [STAC API - Core](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/core)
- **Owner**: @m-mohr

This document is the Language (I18N) Extension to the
[STAC API specification](https://github.com/radiantearth/stac-api-spec),
which defines how to make multi-lingual STAC APIs available.

The focus of this extension is to add API specific behavior on top of the
[language extension for STAC itself](https://github.com/stac-api-extensions/language).
So there's a *STAC* Language extension **and** a *STAC API* Language extension.

This specification aligns as much as possible with
[OGC API - Features](http://docs.opengeospatial.org/DRAFTS/17-069r4.html#string_i18n) and 
[RFC 8288 (Web Linking)](https://www.rfc-editor.org/rfc/rfc8288.html).

This extension applies to all endpoints defined by the STAC API specification and STAC API extension specifications
that may contain linguistic text or links to resources that may contain linguistic text.

## HTTP Headers

The main mechanism for APIs to handle languages is
[HTTP content negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation):

1. The client states its language preferences in the
   [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) header of a request.
   Additionally, servers can support [a query parameter `language`](#query-parameter).
2. The server chooses the language that best matches one of the requested languages and the capabilities of the server.
   The response
   - contains linguistic text in the chosen language, and
   - informs the client of the choice with the `Content-Language` header.

Generally, each language identifier MUST be a valid `Language-Tag` as specified in [RFC 5646](https://www.rfc-editor.org/rfc/rfc5646).

**Example:**

```http
GET / HTTP/2
Host: stac-api.example.com
Accept: application/json,text/json,application/geo+json;q=0.9
Accept-Language: de-DE,de;q=0.9,en-US;q=0.8,en;q=0.7,fr;q=0.3
```

## Query Parameter

In addition to the [header-based approach](#http-headers), servers MAY support a query parameter `language`.

If provided, the `language` query parameter MUST be at least valid `Language-Tag` as specified in [RFC 5646](https://www.rfc-editor.org/rfc/rfc5646).
The query parameter follows the exact same schema as the `Accept-Language` header,
so you can define a list of prioritized languages, e.g. `de-DE,de;q=0.9,en-US;q=0.8,en;q=0.7,fr;q=0.3`.

**Example:**

```http
GET /?language=de HTTP/2
Host: stac-api.example.com
```

**Note:** If both the header `Accept-Language` and the query parameter `language` are provided by a client,
it is recommended to use the values provided in the query parameter.
This is recommended to avoid issues with some clients (such as a web browser) that may send headers
that are not under control of a user.
Ultimately, it's up to the implementation to decide which source it prioritizes.

## Response Body

The response body SHOULD inform clients about the language of the resource and any resource it links to.
For this, it implements the [STAC language extension](https://github.com/stac-api-extensions/language).

### Example

This example request and response is based on the
[example landing page for STAC API - Core](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/core#example-landing-page-for-stac-api---core)
and shows the usage of this extension.

**Header:**

```http
HTTP/2 200 OK
Server: stac-api.example.com
Vary: Accept, Accept-Language
Content-Type: application/json; charset=utf-8
Content-Language: en-US
```

#### Body

```json
{
    "stac_version": "1.0.0",
    "stac_extensions": [
      "https://stac-extensions.github.io/language/v1.0.0/schema.json"
    ],
    "id": "example-stac",
    "title": "A simple STAC API Example",
    "description": "This Catalog aims to demonstrate the a simple landing page",
    "type": "Catalog",
    "language": {
        "code": "en-US",
        "name": "English (United States)"
    },
    "languages": [
        {
            "code": "de",
            "name": "Deutsch",
            "alternate": "German"
        }
    ],
    "conformsTo" : [
        "https://api.stacspec.org/v1.0.0-rc.2/core",
        "https://api.stacspec.org/v1.0.0-rc.2/language"
    ],
    "links": [
        {
            "rel": "self",
            "type": "application/json",
            "href": "https://stac-api.example.com",
            "hreflang": "en-US"
        },
        {
            "rel": "root",
            "type": "application/json",
            "href": "https://stac-api.example.com",
            "hreflang": "en-US"
        },
        {
            "rel": "service-desc",
            "type": "application/vnd.oai.openapi+json;version=3.0",
            "href": "https://example.com/api",
            "hreflang": "en"
        },
        {
            "rel": "service-doc",
            "type": "text/html",
            "href": "https://example.com/api.html",
            "hreflang": "en-US"
        },
        {
            "rel": "service-doc",
            "type": "text/html",
            "href": "https://example.com/de/api.html",
            "hreflang": "de"
        },
        {
            "rel": "alternate",
            "type": "application/json",
            "href": "https://example.com/?language=de",
            "hreflang": "de"
        }
    ]
}
```
