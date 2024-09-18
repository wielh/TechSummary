# RedoclyCli


## overview

Redocly CLI 是一个开源的命令行工具，用于生成、验证、格式化和托管 OpenAPI 规范文档，常用于开发者构建和管理 API 文档。它简化了 OpenAPI 规范的处理，提供了 API 文档的实时预览、验


## 主要功能

+ API 文档生成和预览：你可以使用 Redocly CLI 生成带有可交互界面的 API 文档。支持通过 redocly preview-docs 实时预览 OpenAPI 规范文档。

+ OpenAPI 规范的验证和修复：redocly lint 可以验证 OpenAPI 规范，检测是否符合标准，提示不合规范的部分。支持格式化和修复 OpenAPI 规范，确保文档的可读性和一致性。

+ 文档托管：可以将生成的 API 文档部署到 Redocly 的托管平台上，也可以导出 HTML 文件进行本地或其他平台的托管。

## 指令

用 docker 執行
```
docker run --rm -v $PWD:/spec redocly/cli <參數>
```

yaml to html 
```
redocly build-docs openapi.yaml --output ./docs
```