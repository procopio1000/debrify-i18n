# Auditoria V3 — Debrify i18n

**Data:** 2026-09-25  
**Projeto de planejamento:** procopio1000/debrify-i18n  
**Upstream auditado:** varunsalian/debrify  
**Release verificada:** v0.10.0-beta.1  
**Commit verificado:** 9619c10b06ee919cacbe996b30be7739dc09c6d6  
**Toolchain verificado:** Flutter 3.44.8  
**Plano resultante:** PLANO_MESTRE_V3.md

## Conclusão

A V2 já era ampla, mas ainda deixava algumas decisões críticas para a implementação. A V3 converte essas áreas em contratos verificáveis e corrige dois blockers que poderiam aparecer imediatamente no primeiro PR.

O objetivo desta auditoria não é prometer uma prova matemática de que nenhum literal dinâmico jamais escapará em runtime. A V3 substitui essa promessa por cobertura reproduzível: inventário por reachability, scanners, parity checks, testes de runtime, build matrix, pseudo-locales, revisão manual e allowlist versionada.

## Baseline verificada

No commit auditado:

- o workflow de release usa Flutter 3.44.8;
- o app declara `intl ^0.19.0`;
- ainda não declara `flutter_localizations`;
- não há infraestrutura ARB/AppLocalizations do produto;
- o build publica Android, macOS, Windows, iOS, tvOS, Linux x64 e Linux arm64;
- há código local/vendorizado em `packages/` que participa do runtime;
- o recorte principal de runtime auditado contém 1.574 arquivos, incluindo 971 Dart e 384 XML;
- o Android possui duas Activities nativas principais de player e vários Services/Receivers;
- há superfícies próprias em iOS, tvOS/Top Shelf, macOS, Windows, Linux e Web.

A contagem de 1.574 arquivos é apenas baseline do commit auditado e deve ser recalculada quando o upstream mudar.

## Achados críticos

### 1. Bloqueio de dependência: flutter_localizations x intl

No Flutter 3.44.8, o pacote SDK `flutter_localizations` fixa:

    intl: 0.20.2

O Debrify ainda possui:

    intl: ^0.19.0

Adicionar `flutter_localizations` sem atualizar a dependência direta de `intl` pode impedir `flutter pub get`.

**Correção incorporada na V3:**

- atualizar `intl` no PR A para constraint compatível com 0.20.2;
- não usar dependency override;
- validar lockfile e usos existentes de DateFormat;
- transformar isso em Gate 0 de CI.

### 2. synthetic-package está obsoleto no toolchain real

O comando `gen-l10n` do Flutter 3.44.8 marca `synthetic-package` como removido:

- `true` é rejeitado;
- `false` não tem efeito e gera warning.

A V2 ainda mostrava `synthetic-package: false` no `l10n.yaml`.

**Correção incorporada na V3:** remover a chave e falhar em warnings inesperados de geração.

### 3. Android background precisa de locale sem Flutter

Foram confirmadas superfícies user-facing em componentes que podem executar sem engine Flutter:

- `MediaStoreDownloadService`;
- `LiveRecordingService`;
- `RecordingAlarmReceiver`;
- notification channels/actions/summaries.

Um bridge baseado apenas em Activity/Intent não cobre cold-start, alarmes ou process death.

**Correção incorporada na V3:** `NativeLocaleStore`, espelho nativo read-only da autoridade Dart, com fallback seguro e teste após process kill/reboot.

### 4. Lógica nativa depende de frases inglesas

Foram confirmados branches como:

- `title == "Download complete"`;
- `title.startsWith("Preparing")`;
- `title.startsWith("Retrying")`;
- `title.startsWith("Saved")`;
- `title.startsWith("Starting")`;
- `title.startsWith("Reconnecting")`.

Traduzir essas strings diretamente mudaria comportamento.

**Correção incorporada na V3:** migrar antes para enums/reason codes e mapear estado -> string resource apenas na borda de apresentação.

### 5. XML Android é uma superfície grande, não um detalhe

O projeto tem dezenas de layouts nativos próprios de TV/player com literais, incluindo exemplos como:

- Play / Pause;
- Next Title;
- Jump / Jump to channel;
- Start Over;
- Audio / Subs;
- SUBTITLE SETTINGS;
- Channels;
- Back;
- NOW;
- LIVE;
- instruções de D-pad.

Também há `contentDescription` literal.

**Correção incorporada na V3:**

- scanner para `android:text`, `contentDescription`, hint e similares;
- migração para `@string` / plurals;
- zero literal user-facing fora de allowlist;
- resource parity default x pt-BR.

### 6. SourceSheet concentra vários tipos de dívida i18n

No mesmo componente foram confirmados:

- `label == 'All sources'`;
- plural manual `N sources`;
- `toUpperCase()` em copy;
- semanticLabel `Playing`;
- estados Fetch/failed/empty ingleses;
- decimal de file size manual.

**Correção incorporada na V3:** tratar SourceSheet como unidade atômica, não como substituição isolada de Text literals.

### 7. Formatters estão espalhados

Além de `lib/utils/formatters.dart`, há `DateFormat` direto em outras superfícies como main/settings/sync.

**Correção incorporada na V3:** inventariar chamadas de DateFormat, NumberFormat, String.format, toStringAsFixed e concatenação user-facing em todo o runtime.

### 8. Apple possui UI própria e UI controlada pelo SO

Foram confirmadas strings de sistema em:

- `ios/Runner/Info.plist` — NSLocalNetworkUsageDescription;
- `macos/Runner/Info.plist` — NSLocalNetworkUsageDescription.

Também foi confirmado que o menu macOS em `Base.lproj/MainMenu.xib` contém grande quantidade de copy inglesa.

O ponto arquitetural importante é que `InfoPlist.strings` e permission prompts são escolhidos pelo bundle/idioma do sistema/per-app language. Um seletor interno Flutter não consegue reescrever isso de forma suportada em runtime.

**Correção incorporada na V3:**

- ownership matrix por superfície;
- localização completa do bundle;
- teste por idioma do sistema/per-app;
- macOS MainMenu localizado;
- nenhuma promessa falsa de que override Dart controla prompt do SO.

### 9. tvOS Top Shelf é out-of-process

A extensão pode rodar sem Flutter. Seus títulos/sinopses atuais são majoritariamente dados externos e devem continuar assim.

**Correção incorporada na V3:** qualquer copy própria futura da extensão usa bundle localization ou snapshot já localizado; nunca depende de MethodChannel ativo.

### 10. Windows installer continua somente em inglês

`windows/installer.iss` declara somente o idioma inglês e possui custom copy inglesa.

`windows/runner/Runner.rc` ainda possui metadata de template como:

    CompanyName = com.example

**Correção incorporada na V3:**

- suporte PT-BR no installer;
- custom messages/tasks localizados;
- teste installer en/PT-BR;
- remoção de metadata template.

### 11. Web precisa refletir locale no documento

Foram confirmados placeholders `A new Flutter project` em index/manifest e ausência de contrato runtime para `lang`/`dir`.

**Correção incorporada na V3:** `WebLocaleBridge` para sincronizar BCP-47 e direção do documento.

### 12. Voice/input é uma autoridade própria

O Android já aceita BCP-47 em `RecognizerIntent.EXTRA_LANGUAGE`.

**Correção incorporada na V3:** não amarrar automaticamente speech locale ao App language. A preferência de input/voz é separada e pode continuar system-default.

### 13. Regional matching exige política explícita

Com apenas `pt-BR` disponível, um fallback genérico por languageCode pode transformar `pt-PT` em Brazilian Portuguese silenciosamente.

**Correção incorporada na V3:** resolver customizado, com `pt-PT` não caindo automaticamente em `pt-BR`, e casos explícitos para `pt`, `pt_BR`, `en-US` e `en-GB`.

### 14. packages/ não pode ser ignorado cegamente

O Debrify possui packages locais/vendorizados usados pelo runtime. Um scanner que considere apenas `lib/` e plataformas pode perder copy empacotada; um scanner que inclua tudo sem classificação gera ruído de examples/tests.

**Correção incorporada na V3:** scan por reachability + classificações GENERATED_CODE, TEST_ONLY, SYSTEM_OWNED_UI e THIRD_PARTY_OWNED_UI.

### 15. Bidi isolation faltava como requisito

RTL não é apenas inverter paddings. Dados externos LTR dentro de copy RTL podem quebrar ordem visual.

**Correção incorporada na V3:** isolamento direcional na camada de apresentação para URL/path/filename/release/user data, sem alterar o valor persistido.

## Achados de qualidade de shell

A auditoria também confirmou itens que não são “tradução de string” pura, mas afetam a qualidade do release localizado:

- Web ainda possui metadata do template Flutter;
- Windows Runner.rc ainda possui `com.example`;
- macOS menu nativo permanece Base/English;
- Android strings.xml contém somente uma fração das strings nativas reais.

Esses itens entram na fase de native shell hardening porque um release “PT-BR completo” não pode deixar superfícies oficiais de instalação/sistema em estado de template.

## Decisões V3

A V3 estabelece como canônicos:

1. Flutter 3.44.8 + intl 0.20.2 para a baseline atual.
2. ARB/gen_l10n sem synthetic-package.
3. AppLocaleController como autoridade do app.
4. NativeLocaleStore Android como espelho, nunca segunda autoridade.
5. Ownership matrix para superfícies OS-owned.
6. ShippingLocales separado de generated locales.
7. Resolver regional customizado/testado.
8. Estado tipado antes de localização em services nativos.
9. Scan por reachability do artifact.
10. Web lang/dir em runtime.
11. Speech/input locale independente.
12. Bidi isolation para dados mistos.
13. Evidência mínima reproduzível por PR.

## Resultado

`PLANO_MESTRE_V3.md` substitui a V2 como especificação canônica.

V1 e V2 ficam preservadas como histórico para permitir comparar decisões e rastrear por que cada hardening foi introduzido.
