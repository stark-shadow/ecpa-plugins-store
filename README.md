# ECPA Plugins Store

ECPA 精选插件目录与自制插件的发布索引仓库。

- `registry.json` — 插件目录（schema_version 1）

## 结构

```json
{
  "schema_version": 1,
  "plugins": [
    {
      "id": "plugin-id",
      "name": "Display Name",
      "description": "English description",
      "author": "author",
      "version": "1.0.0",
      "repository": "https://github.com/owner/repo",
      "logo": "https://...",
      "homepage": "https://...",
      "license": "MIT",
      "tags": ["Interceptor"],
      "locales": {
        "zh-CN": { "description": "中文描述" }
      }
    }
  ]
}
```

## 约定

- 新增插件按 `id` 升序追加，不改动已有条目。
- `version` 必须与对应仓库的 release tag 一致（去掉前缀 `v`）。
- 中英文描述同时提供，`locales.zh-CN.description` 为必填。
