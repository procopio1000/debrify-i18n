# Auditoria V8 — Debrify i18n/l10n

**Data:** 2026-09-25  
**Plano resultante:** `PLANO_MESTRE_V8.md`  
**Upstream:** `varunsalian/debrify`  
**Release revalidada:** `v0.10.0-beta.1`  
**Commit revalidado:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`  
**Tree revalidada:** `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`

## Cobertura da reauditoria

A árvore recursive do commit-alvo foi revalidada com `truncated=false`: 3.199 blobs, 1.736 nos roots first-party de produto e 2.109 ao incluir `packages/`. Também foram confrontados o plano V7, README, arquitetura, matriz, quality gates, workflow/toolchain, armazenamento de preferências Dart/Android e superfícies oficiais externas acionadas pelo app.

## Achados novos

### 1. Gate P ausente no documento canônico

A V7 exigia Gate P na Definition of Done e o descrevia em `AUDITORIA_V7.md` e `CI_QUALITY_GATES.md`, mas a seção 11 do plano canônico terminava no Gate O.

**Correção V8:** Gate P entra no plano mestre e no registro canônico de gates.

### 2. IDs incompatíveis entre plano e CI

O plano usava A–O enquanto o documento de CI usava 1–14 e aliases parciais.

**Risco:** automação/Codex pode executar uma regra sob o ID errado ou considerar um gate concluído por equivalência inexistente.

**Correção V8:** um único namespace normativo: `0, A..Q`. Documentos auxiliares não podem renumerar.

### 3. Ausência de blocker contra drift do upstream

A baseline está correta hoje: o `main` do upstream ainda está em `9619c10…`. Porém “revalidar/rebasear” era orientação, não condição executável.

**Correção V8:** Gate Q + `AUDIT_BASELINE_MANIFEST.json`. Mudança da base expira a prova de completude até o delta ser auditado.

### 4. Contrato de locale amplo demais

`Locale` do Flutter preserva language/script/region; o plano dizia genericamente “persistir BCP-47”, o que poderia sugerir round-trip de variants/extensions/private-use.

**Correção V8:** `ProductLocaleId = language[-Script][-REGION]`, sentinel `system`, rejeição de perda silenciosa e adapters de representação por plataforma.

### 5. Backing store Android pode ser fixado com precisão

Na baseline, `DevicePreferences` encapsula `SharedPreferences.getInstance()`. O Android nativo já lê `FlutterSharedPreferences` e chaves `flutter.*` em múltiplos caminhos.

Versões resolvidas auditadas:

- `shared_preferences 2.5.3`;
- `shared_preferences_android 2.4.10`.

**Contrato baseline para locale:** arquivo `FlutterSharedPreferences`, key `flutter.ui_locale_v1`. O detalhe fica encapsulado no NativeLocaleStore e é revalidado por Gate 0/Q se o backend mudar.

### 6. Locale de website/browser/QR não é o mesmo que App language

O app pode abrir conteúdo Debrify no browser e o QR pode ser lido em outro dispositivo. Nesses casos, browser/SO e dispositivo receptor podem ter locale diferente do app de origem.

**Correção V8:** locale explícito em URL quando suportado; caso contrário landing neutra com selector/fallback documentado. Nunca tratar `Accept-Language` do browser externo como prova do override interno.

### 7. Política geral para caches de apresentação

A V7 tratava caches em pontos específicos, mas não havia invariant global contra guardar frases já localizadas sem locale na chave.

**Correção V8:** cachear dados semânticos por padrão; presentation cache só com ProductLocaleId/invalidation; teste en → pt-BR → en sem restart.

### 8. Locale da árvore de acessibilidade

No Flutter 3.44.8, `Localizations` em nível de aplicação não atribui `localeForSubtree` no `Semantics` que cria; esse atributo é usado pelo caminho não-application-level. Portanto, texto traduzido não basta para provar a language attribution quando App language difere do system locale.

**Correção V8:** inspeção da Semantics tree, wrapper `Semantics(localeForSubtree: effectiveLocale)` quando necessário e smoke real de screen reader com system/app locales diferentes.

## Ajustes documentais

- V8 incorpora P e Q ao plano canônico.
- Quality gates passam a usar IDs idênticos ao plano.
- Arquitetura/matriz deixam versão histórica apenas nos arquivos `AUDITORIA_V*.md`; documentos normativos são temáticos.
- README passa a apontar V8 como fonte canônica.
- O manifest machine-readable fixa SHA/tree, contagens e dependências que afetam contratos nativos.

## Resultado

A V8 não altera o objetivo funcional da V7; ela fecha **ambiguidade executável**. O plano deixa de depender de equivalência implícita entre documentos e de uma baseline “lembrada” por humanos, e passa a tornar essas premissas verificáveis por automação.
