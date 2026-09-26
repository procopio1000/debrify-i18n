# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Estado atual

**Plano canônico:** `PLANO_MESTRE_V9.md`

A V9 foi re-auditada contra a mesma árvore exata do upstream em 2026-09-25; no momento da auditoria, o `main` upstream continuava idêntico ao commit auditado. Ela preserva V1–V8 e fecha ambiguidades restantes de calendário/tempo, concorrência assíncrona e autoridade de locale.

Principais reforços V9:

- taxonomia temporal separa data/hora humana de timecode, protocolo, filename, diagnóstico e regras de calendário de provider;
- hotspots reais de calendário/formatting ficam registrados em `docs/AUDIT_HOTSPOTS_V9.json`;
- `localeEpoch` impede completion assíncrona do locale anterior de sobrescrever presentation state atual;
- `LocaleFallbackPolicy` fica independente de `ShippingLocales`;
- primeiro release pt-BR não cria segunda autoridade via Android per-app language;
- search normalization ganha conformance NFC/NFD/combining marks;
- collation real deixa de ser confundida com lowercase/search-fold;
- Semantics language attribution passa a ser scoped por ownership;
- locale flip vira contrato presentation-only;
- runtime roots sob `packages/**` ganham regra operacional explícita de scan.

O registro de gates continua estável em `0, A..Q`; a V9 fortalece gates existentes sem renumerá-los.

V1–V8 permanecem como histórico/audit trail e não substituem a V9.

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

- `PLANO_MESTRE_V9.md` — fonte canônica de implementação
- `docs/ARQUITETURA.md` — arquitetura V9
- `docs/CI_QUALITY_GATES.md` — gates V9 com IDs canônicos
- `docs/MATRIZ_TESTES.md` — matriz V9
- `docs/AUDIT_BASELINE_MANIFEST.json` — baseline machine-readable
- `docs/AUDIT_HOTSPOTS_V9.json` — hotspots confirmados machine-readable
- `docs/GLOSSARIO_PT_BR.md` — terminologia PT-BR

## Evidência/audit trail

- `docs/AUDITORIA_V9.md` — nona auditoria
- `docs/AUDITORIA_V8.md` — oitava auditoria
- `docs/AUDITORIA_V7.md`
- `docs/AUDITORIA_V6.md`
- `docs/AUDITORIA_V5.md`
- `docs/AUDITORIA_V4.md`
- `docs/AUDITORIA_V3.md`
- `docs/AUDITORIA_V2.md`
- `docs/AUDITORIA_BASELINE.md`
- `PLANO_MESTRE_V1.md` … `PLANO_MESTRE_V8.md` — versões superseded

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
