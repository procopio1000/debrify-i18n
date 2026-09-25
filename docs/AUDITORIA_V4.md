# Auditoria V4 — Debrify i18n

**Data:** 2026-09-25  
**Projeto de planejamento:** procopio1000/debrify-i18n  
**Upstream auditado:** varunsalian/debrify  
**Release verificada:** v0.10.0-beta.1  
**Commit verificado:** 9619c10b06ee919cacbe996b30be7739dc09c6d6  
**Toolchain reconfirmado:** Flutter 3.44.8  
**Plano resultante:** PLANO_MESTRE_V4.md

## Conclusão

A V3 já era forte e implementation-ready para a arquitetura principal. A V4 fecha uma classe diferente de lacunas: texto user-facing que não nasce de ARB, Android resources ou widgets `Text` tradicionais.

A reauditoria cruzou o plano com o runtime atual do upstream, loaders de assets, configuração remota, notification builders e sinks de renderização de texto. O baseline upstream continua exatamente no commit auditado, portanto os achados abaixo não são consequência de drift de versão.

## Baseline reconfirmada

Em 2026-09-25, `varunsalian/debrify` continua em:

    v0.10.0-beta.1
    9619c10b06ee919cacbe996b30be7739dc09c6d6

O Flutter SDK usado pela baseline continua 3.44.8.

O SDK `flutter_localizations` dessa versão declara:

    intl: 0.20.2

Portanto permanece correta a decisão da V3/V4 de atualizar a dependência direta `intl ^0.19.0` do Debrify no mesmo PR que introduzir `flutter_localizations`.

## Novos achados V4

### 1. SupportRemoteConfig injeta copy oficial fora do ARB

Arquivos confirmados:

    assets/config/app_remote_config.json
    lib/services/support_remote_config_service.dart
    lib/main.dart
    lib/screens/settings_screen.dart

O fallback empacotado contém copy como:

    Support Debrify
    Help fund development with a donation
    Donate on Ko-fi

O service também possui fallbacks hardcoded como `Sponsor`.

`main.dart` e Settings consomem diretamente os campos remotos.

**Risco:** ARB PT-BR pode estar 100% completo e a interface ainda exibir inglês.

**Correção V4:**

- fixed product copy vai para ARB;
- providers/URLs permanecem dados;
- campanha dinâmica ganha schema locale-aware versionado;
- fallback exact BCP-47 -> regra explícita -> en;
- fallback inglês mantém robustez, mas conta como lacuna de completeness para um shipping locale;
- cache não pode congelar a variante do locale anterior.

### 2. Catálogo oficial de engines é uma dependência i18n de runtime

O onboarding/engine flow consome catálogo remoto `metadata.yaml`.

Descrições editoriais controladas pelo produto não devem ser tratadas da mesma forma que texto arbitrário de um engine importado pelo usuário.

**Correção V4:** introduzir a classe `PRODUCT_CONTROLLED_REMOTE_CATALOG`.

- descrição oficial: locale-aware schema ou mapper local versionado;
- brand/name técnico: preservado quando apropriado;
- engine de terceiro: `THIRD_PARTY_EXTERNAL_DATA`;
- fallback inglês do catálogo oficial aparece no completeness report.

### 3. background_downloader possui notification copy inglesa fora do caminho Android nativo

Arquivo confirmado:

    lib/services/download_service.dart

No caminho não-Android:

    TaskNotification('Downloading', '{filename}')
    TaskNotification('Download complete', '{filename}')
    TaskNotification('Download failed', '{filename}')
    TaskNotification('Download paused', '{filename}')

**Correção V4:** incluir plugin notification builders no inventário, nos testes e no DoD em toda plataforma onde essa copy for apresentada ao usuário.

`{filename}` permanece dado externo/placeholder.

### 4. Text.rich/TextSpan escapam de scanner superficial

Exemplos confirmados:

    lib/widgets/series_browser.dart              -> Resume ·
    lib/screens/search/search_sources.dart       -> Season
    lib/screens/addons/addon_hub_screen.dart     -> Community addons from
    lib/widgets/aggregated_search_results.dart   -> Tap to search

**Correção V4:** `Text.rich`, `RichText`, `TextSpan` e `InlineSpan` passam a ser sinks explícitos do Gate D.

### 5. Custom painters desenham copy user-facing

Exemplos confirmados em launch idents:

    lib/widgets/launch/blueprint_ident.dart
    lib/widgets/launch/swiss_ident.dart
    lib/widgets/launch/constellation_ident.dart

Há texto como:

    PLAY · ANYTHING
    LAUNCH
    DEBRID · TORRENT · IPTV

**Correção V4:** `TextPainter`/Canvas entram no inventário. Cada ocorrência precisa ser localizada ou classificada como `BRAND_ART_DIRECTION` com justificativa.

### 6. Runtime assets precisam fazer parte da reachability

`rootBundle.loadString` já é usado para configuração e outros dados.

Apenas escanear Dart/Kotlin/XML não prova cobertura de copy que chega à UI por JSON/YAML/Markdown/CSV.

**Correção V4:** o inventário passa a construir também um grafo de assets alcançáveis por:

- pubspec/asset manifest;
- `rootBundle`;
- loaders próprios;
- fallback assets;
- remote config.

### 7. Hardcoded directionality exige gate dedicado

Foram confirmados `TextDirection.ltr` em código de produção. Parte é intencional — branding, animação física ou controles de mídia — mas há também uso app-owned em medição de texto, como em:

    lib/screens/catalog_item_detail_screen.dart

**Correção V4:** novo Gate M classifica toda directionality física versus semântica e reporta usos não justificados.

### 8. Rich text precisa permitir reordenação por idioma

Traduzir apenas fragmentos preservando a ordem inglesa pode produzir gramática incorreta.

**Correção V4:** mensagens com spans estilizados/clicáveis usam placeholders semânticos e renderer reorder-safe, mantendo Semantics e bidi isolation.

### 9. O seletor de idioma precisa usar autônimos

Política V4:

    en     -> English
    pt-BR  -> Português (Brasil)

O nome do destino não deve ficar incompreensível porque foi traduzido para o idioma atualmente ativo.

### 10. “External data” foi refinado

V4 separa:

    LOCAL_PRODUCT_COPY
    REMOTE_PRODUCT_COPY
    PRODUCT_CONTROLLED_REMOTE_CATALOG
    THIRD_PARTY_EXTERNAL_DATA
    USER_DATA
    BRAND/TECHNICAL_TOKEN

Essa distinção evita dois erros opostos:

- tentar traduzir filenames/títulos/EPG arbitrários;
- deixar copy oficial inglesa escapar sob o rótulo genérico “external”.

## Novos gates

A V4 adiciona dois gates transversais:

### Gate L — Remote/runtime product copy

Verifica:

- copy oficial remota sem política de locale;
- fallback assets;
- cache locale-safe;
- catálogo oficial remoto;
- runtime asset copy;
- fallback inglês em shipping locale.

### Gate M — Directionality and inline-text safety

Verifica:

- hardcoded `TextDirection`;
- Alignment/EdgeInsets/Positioned físicos suspeitos;
- directional icons;
- `TextSpan` com ordem rígida;
- pseudo-RTL;
- mixed-direction data.

## Critério de completude V4

O relatório de release passa a separar e somar:

    ARB
    native resources
    remote product copy
    product-controlled remote catalog
    runtime asset copy
    approved external/user data

Assim, “100%” deixa de significar apenas “todas as chaves ARB existem” e passa a significar “toda copy própria alcançável no runtime tem owner, locale policy e evidência”.

## Resultado

`PLANO_MESTRE_V4.md` substitui a V3 como especificação canônica.

V1, V2 e V3 permanecem preservadas como histórico/audit trail.
