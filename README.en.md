# bce-openapi-meta

[中文](./README.md)

BCE OpenAPI metadata repository. Describes Baidu Cloud product OpenAPI definitions and multilingual descriptions in structured JSON.

---

## Project Structure

```
bce-openapi-meta/
├── meta.go               # Go entry point; embeds schema/ and i18n/ into the binary via embed.FS
├── go.mod                # Go module definition
├── schema/               # API action definitions
│   ├── products.json     # Product list with regional endpoint configuration
│   └── <product>/        # One directory per product; one file per action
│       └── <Action>.json
└── i18n/                 # Multilingual descriptions
    ├── zh-CN/
    │   ├── common.json   # Shared parameter descriptions (version, region, marker, etc.)
    │   └── <product>/
    │       └── <product>.json
    └── en-US/
        ├── common.json
        └── <product>/
            └── <product>.json
```

---

## Data Format

### products.json

Lists supported products and their service endpoints per region.

```json
{
  "products": [
    {
      "code": "vpc",
      "protocol": "HTTP",
      "endpoint": {
        "bj": "bcc.bj.baidubce.com",
        "gz": "bcc.gz.baidubce.com"
      }
    }
  ]
}
```

### schema/\<product\>/\<Action\>.json

Describes a single API action: HTTP method, URI template, and parameter definitions. Parameter descriptions are referenced via `desc_key` in the format `<product>.<Action>.<param>` or `common.<param>`.

```json
{
  "name": "CreateVpc",
  "method": "POST",
  "uri": "/v{version}/vpc",
  "desc_key": "vpc.CreateVpc.desc",
  "parameters": [
    {
      "name": "name",
      "type": "String",
      "required": true,
      "position": "BODY",
      "desc_key": "vpc.CreateVpc.name"
    },
    {
      "name": "tags",
      "type": "List",
      "required": false,
      "position": "BODY",
      "desc_key": "vpc.CreateVpc.tags",
      "sub_parameters": [
        {
          "name": "tagKey",
          "type": "String",
          "required": true,
          "desc_key": "vpc.CreateVpc.tags.tagKey"
        }
      ]
    }
  ]
}
```

Valid `position` values: `URL` · `QUERY` · `BODY`

### i18n/\<locale\>/\<product\>/\<product\>.json

Stores parameter descriptions keyed by `<Action>.<param>` for each locale.

```json
{
  "CreateVpc": {
    "desc": "Create a VPC",
    "name": "VPC name",
    "cidr": "CIDR block for the VPC"
  }
}
```

---

## Supported Products

| Code | Product | Actions |
|---|---|---|
| `vpc` | Virtual Private Cloud | 84 |

---

## Usage

This module is used as a Go library. All metadata is embedded into the binary at compile time via `embed.FS`, requiring no runtime file dependencies.

```go
import bceopenapimeta "github.com/baidubce/bce-openapi-meta"

// Read a schema file
data, err := bceopenapimeta.FS.ReadFile("schema/vpc/CreateVpc.json")

// Read i18n descriptions
i18n, err := bceopenapimeta.FS.ReadFile("i18n/en-US/vpc/vpc.json")
```

---

## Adding a New Product

1. Add a product entry (`code`, `protocol`, `endpoint`) to the `products` array in `schema/products.json`.
2. Create `schema/<product>/` and add one JSON file per API action.
3. Create `i18n/zh-CN/<product>/` and `i18n/en-US/<product>/` directories with the corresponding description files.
4. Common parameters (`version`, `region`, `marker`, etc.) can reuse `desc_key` values from `common.json` without duplication.
