# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Estado atual

**Plano canônico:** `PLANO_MESTRE_V8.md`

A V8 foi re-auditada contra a árvore exata do upstream em 2026-09-25. Ela preserva todos os hardenings V1–V7 e fecha as últimas ambiguidades executáveis encontradas entre plano, CI e runtime nativo.

Principais reforços V8:

- Gate P passa a existir também no plano canônico, não apenas nos documentos auxiliares;
- um único registro de gates: `0, A..Q`;
- Gate Q bloqueia drift não auditado do upstream;
- `docs/AUDIT_BASELINE_MANIFEST.json` fixa baseline de forma machine-readable;
- `ProductLocaleId = language[-Script][-REGION]` evita prometer round-trip de BCP-47 que o `Locale` do produto não preserva;
- contrato Android baseline exato: `FlutterSharedPreferences / flutter.ui_locale_v1`;
- presentation caches precisam ser locale-keyed/invalidation-safe;
- links/QR oficiais distinguem App language de browser/system/outro dispositivo.

V1–V7 permanecem como histórico/audit trail e não substituem a V8.

## Alvo verificado

- Upstream: `varunsalian/debrify`
- Baseline: `v0.10.0-beta.1`
- Commit auditado: `9619c10b06ee919cacbe996b30be7739dc09c6d6`
- Tree auditada: `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`
- Recursive tree: completa (`truncated=false`)
- Primeiro locale completo: `pt-BR`
- Template canônico: `en`

Contagem estrutural reproduzida na V8:

- 3.199 blobs totais;
- 1.736 arquivos nos roots first-party de produto;
- 2.109 ao incluir `packages/`.

Contagem estrutural não significa que todo arquivo contém UI; reachability e ownership continuam obrigatórios.

## Arquivos normativos

- `PLANO_MESTRE_V8.md` — fonte canônica de implementação
- `docs/ARQUITETURA.md` — arquitetura V8
- `docs/CI_QUALITY_GATES.md` — gates V8 com IDs canônicos
- `docs/MATRIZ_TESTES.md` — matriz V8
- `docs/AUDIT_BASELINE_MANIFEST.json` — baseline machine-readable
- `docs/GLOSSARIO_PT_BR.md` — terminologia PT-BR

## Evidência/audit trail

- `docs/AUDITORIA_V8.md` — oitava auditoria
- `docs/AUDITORIA_V7.md`
- `docs/AUDITORIA_V6.md`
- `docs/AUDITORIA_V5.md`
- `docs/AUDITORIA_V4.md`
- `docs/AUDITORIA_V3.md`
- `docs/AUDITORIA_V2.md`
- `docs/AUDITORIA_BASELINE.md`
- `PLANO_MESTRE_V1.md` … `PLANO_MESTRE_V7.md` — versões superseded

## O que o plano cobre

A infraestrutura proposta separa App language de:

- idioma de metadados;
- áudio;
- legendas;
- região;
- input/voz;
- identidade/protocolo.

E cobre, por reachability:

- Flutter UI e múltiplos roots/bootstrap paths;
- SettingsRows/Settings Search;
- rich text/custom painting;
- player Flutter;
- Android TV native players;
- Services/Receivers/notifications/PiP;
- iOS/tvOS/Top Shelf/macOS;
- Windows/Linux/Web/PWA;
- runtime assets e remote product copy;
- build-generated product copy;
- official product content;
- accessibility/RTL/pseudo locales;
- Unicode/grapheme/list composition;
- human numeric input;
- cross-runtime/cross-device reason codes;
- packaged artifact inspection;
- real-device runtime smoke;
- rollout/rollback;
- baseline drift e audit freshness.

## Registro canônico de gates

    0
    A B C D E F G H I J K L M N O P Q

Nenhum documento normativo pode renumerar ou criar alias desses IDs.

## Regra central

Texto localizado nunca funciona como:

- ID;
- valor persistido canônico;
- condição de lógica;
- protocolo;
- cache key semântica.

A afirmação de completude vale somente para o SHA/tree auditado. Se o upstream muda, Gate Q precisa revalidar o delta antes que a cobertura seja considerada vigente.
