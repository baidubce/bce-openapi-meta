# bce-openapi-meta

[English](./README.en.md)

BCE OpenAPI 元数据库，以结构化 JSON 描述百度云各产品 OpenAPI 的接口定义与多语言文案。

---

## 项目结构

```
bce-openapi-meta/
├── meta.go               # Go 入口，通过 embed.FS 将 schema/ 和 i18n/ 嵌入二进制
├── go.mod                # Go 模块定义
├── schema/               # API 接口定义
│   ├── products.json     # 产品列表及各 Region Endpoint 配置
│   └── <product>/        # 按产品分目录，每个文件对应一个接口
│       └── <Action>.json
└── i18n/                 # 多语言文案
    ├── zh-CN/
    │   ├── common.json   # 公共参数描述（version、region、marker 等）
    │   └── <product>/
    │       └── <product>.json
    └── en-US/
        ├── common.json
        └── <product>/
            └── <product>.json
```

---

## 数据格式

### products.json

描述已接入的产品列表及各地域的服务 Endpoint。

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

描述单个 API 接口的 HTTP 方法、路径及参数定义。参数描述通过 `desc_key` 引用 i18n 文案，格式为 `<product>.<Action>.<param>` 或 `common.<param>`。

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

参数位置（`position`）取值：`URL` · `QUERY` · `BODY`

### i18n/\<locale\>/\<product\>/\<product\>.json

以 `<Action>.<param>` 为键存储各语言的参数描述文案。

```json
{
  "CreateVpc": {
    "desc": "创建 VPC",
    "name": "VPC 名称",
    "cidr": "VPC 的 CIDR 地址块"
  }
}
```

---

## 已支持产品

| 产品代码 | 产品名称 | 接口数量 |
|---|---|---|
| `vpc` | 虚拟私有云 | 84 |

---

## 使用方式

该模块作为 Go 库被引用，通过 `embed.FS` 在编译时将所有元数据打包进二进制，无需运行时文件依赖。

```go
import bceopenapimeta "github.com/baidubce/bce-openapi-meta"

// 读取 schema 文件
data, err := bceopenapimeta.FS.ReadFile("schema/vpc/CreateVpc.json")

// 读取 i18n 文案
i18n, err := bceopenapimeta.FS.ReadFile("i18n/zh-CN/vpc/vpc.json")
```

---

## 如何新增产品

1. 在 `schema/products.json` 的 `products` 数组中添加产品条目（`code`、`protocol`、`endpoint`）。
2. 创建 `schema/<product>/` 目录，按接口名称添加 JSON 文件。
3. 创建 `i18n/zh-CN/<product>/` 和 `i18n/en-US/<product>/` 目录，添加对应的文案文件。
4. 公共参数（`version`、`region`、`marker` 等）直接复用 `common.json` 中的 `desc_key`，无需重复定义。
