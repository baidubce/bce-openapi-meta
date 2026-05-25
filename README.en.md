# bce-openapi-meta

[中文](./README.md)

BCE OpenAPI metadata repository. Describes Baidu Cloud product OpenAPI definitions and multilingual descriptions in structured JSON.

---

## Project Structure

```
bce-openapi-meta/
├── schema/               # API action definitions
│   ├── products.json     # Product list with regional endpoint configuration
│   └── <product>/        # One directory per product; one file per action
│       └── <Action>.json
└── i18n/                 # Multilingual descriptions
    ├── zh-CN/
    │   ├── common.json          # Shared parameter descriptions (version, region, marker, etc.)
    │   └── <product>/
    │       ├── version.json     # Product action index (action names and summaries)
    │       └── <Action>.json    # Per-action parameter descriptions
    └── en-US/
        ├── common.json
        └── <product>/
            ├── version.json
            └── <Action>.json
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

### i18n/\<locale\>/\<product\>/version.json

Product action index listing the title and summary of each action.

```json
{
  "version": "v1",
  "style": "rpc",
  "apis": {
    "CreateVpc": {
      "title": "Create VPC",
      "summary": "Create a Virtual Private Cloud"
    }
  }
}
```

### i18n/\<locale\>/\<product\>/\<Action\>.json

Per-action multilingual parameter descriptions, keyed by parameter name.

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

This is a pure data repository. It is consumed by bce-cli as a git submodule and embedded into the binary at compile time via `go:embed`.

```bash
# Add to bce-cli
git submodule add <repo-url> internal/meta/bce-openapi-meta

# Update to latest metadata
git submodule update --remote internal/meta/bce-openapi-meta
```

---

## Adding a New Product

1. Add a product entry (`code`, `protocol`, `endpoint`) to the `products` array in `schema/products.json`.
2. Create `schema/<product>/` and add one JSON file per API action.
3. Create `i18n/zh-CN/<product>/` and `i18n/en-US/<product>/` directories with a `version.json` index and one description file per action.
4. Common parameters (`version`, `region`, `marker`, etc.) can reuse `desc_key` values from `common.json` without duplication.
