# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Estado atual

**Plano canônico:** `PLANO_MESTRE_V10.md`

A V10 foi re-auditada contra a mesma árvore exata do upstream em 2026-09-26; o `main` upstream continua idêntico ao commit auditado. Ela preserva V1–V9 e fecha blind spots de copy indireta em enums/models/registries, presentation formatting fora da borda visual e prova formal de cobertura contra a Git tree.

Principais reforços V10:

- Gate D passa a rastrear copy app-owned em enums, extension getters, models/services, registries e option tables até o sink;
- `CalendarTimeFormat` entra como hotspot explícito: labels deixam o enum e AM/PM manual deixa o formatter;
- Stremio TV now-playing deixa de materializar `Ended / Ends at ...` dentro do model;
- Debrify TV stats e o TV time picker entram no inventário temporal/UI;
- métricas compactas de YouTube/Reddit/Lemmy passam a separar número canônico de presentation formatting;
- Settings e Settings Search compartilham o mesmo localized mapper para options persistidas;
- `RuntimeSurfaceUniverse == ClassifiedReachable ∪ ExcludedWithEvidence` transforma “100%” em closure verificável da baseline;
- nenhum Gate R é criado: D/E/F/P/Q são endurecidos mantendo `0, A..Q`.

O registro de gates continua estável em `0, A..Q`; a V9 fortalece gates existentes sem renumerá-los.

V1–V9 permanecem como histórico/audit trail e não substituem a V10.

## Alvo verificado

- Upstream: `varunsalian/debrify`
- Baseline: `v0.10.0-beta.1`
- Commit auditado: `9619c10b06ee919cacbe996b30be7739dc09c6d6`
- Tree auditada: `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`
- Recursive tree: completa (`truncated=false`)
- Primeiro locale completo: `pt-BR`
- Template canônico: `en`

Contagem estrutural revalidada na V10 (mesma tree):

- 3.199 blobs totais;
- 1.736 arquivos nos roots first-party de produto;
- 2.109 ao incluir `packages/`.

Contagem estrutural não significa que todo arquivo contém UI; reachability e ownership continuam obrigatórios.

## Arquivos normativos

- `PLANO_MESTRE_V10.md` — fonte canônica de implementação
- `docs/ARQUITETURA.md` — arquitetura V10
- `docs/CI_QUALITY_GATES.md` — gates V10 com IDs canônicos
- `docs/MATRIZ_TESTES.md` — matriz V10
- `docs/AUDIT_BASELINE_MANIFEST.json` — baseline machine-readable
- `docs/AUDIT_HOTSPOTS_V10.json` — hotspots V10 confirmados machine-readable
- `docs/AUDIT_HOTSPOTS_V9.json` — hotspots V9 herdados
- `docs/GLOSSARIO_PT_BR.md` — terminologia PT-BR

## Evidência/audit trail

- `docs/AUDITORIA_V10.md` — décima auditoria
- `docs/AUDITORIA_V9.md` — nona auditoria
- `docs/AUDITORIA_V8.md` — oitava auditoria
- `docs/AUDITORIA_V7.md`
- `docs/AUDITORIA_V6.md`
- `docs/AUDITORIA_V5.md`
- `docs/AUDITORIA_V4.md`
- `docs/AUDITORIA_V3.md`
- `docs/AUDITORIA_V2.md`
- `docs/AUDITORIA_BASELINE.md`
- `PLANO_MESTRE_V1.md` … `PLANO_MESTRE_V9.md` — versões superseded

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
- baseline drift e audit freshness;
- declarative presentation dataflow;
- coverage closure exata contra a Git tree.

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
