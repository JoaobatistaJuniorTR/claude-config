---
name: Parent POM pattern no taxsami_artifacts
description: O repositorio taxsami_artifacts usa parent pom - ao bumpar versao de modulo, atualizar TANTO o pom.xml do modulo QUANTO a property no pom.xml raiz
type: project
originSessionId: 68a80cd7-2f31-438e-82a6-74920f8c1155
---
O repositorio `taxsami_artifacts` usa um **parent pom pattern**. O `pom.xml` raiz define properties com as versoes de todos os modulos:

```xml
<taxone-integration.version>X.Y.Z</taxone-integration.version>
```

Ao fazer bump de versao em qualquer modulo:
1. Atualizar o `pom.xml` do modulo (ex: `taxone-integration/pom.xml`)
2. Atualizar a property correspondente no `pom.xml` raiz (ex: `<taxone-integration.version>`)

**Why:** O parent pom gerencia as versoes centralmente. Se atualizar apenas o modulo e nao o parent, a build vai usar a versao antiga.

**How to apply:** Sempre que fizer bump de versao em qualquer modulo dentro de taxsami_artifacts, lembrar de atualizar os DOIS pom.xml.
