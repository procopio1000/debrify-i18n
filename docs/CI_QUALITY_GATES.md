# Quality Gates i18n — V8

Este documento é normativo e usa exatamente os mesmos IDs de `PLANO_MESTRE_V8.md`.

**Registro canônico:** `0, A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q`.

É proibido manter aliases como “Gate 1”, “Gate 12 / Gate L” ou renumerar gates em scripts/documentação.

## Gate 0 — dependency/toolchain preflight

Baseline auditada:

    Flutter 3.44.8
    intl resolvido antes da fundação i18n = 0.19.0
    flutter_localizations alvo -> intl 0.20.2
    shared_preferences = 2.5.3
    shared_preferences_android = 2.4.10

Run:

    flutter --version
    flutter pub get
    flutter pub deps
    flutter gen-l10n

Fail if:

- Flutter diverge da versão de baseline sem reauditoria;
- `intl` não resolve 0.20.2 depois de adicionar `flutter_localizations`;
- `dependency_overrides` mascara conflito;
- `l10n.yaml` contém `synthetic-package`;
- gen-l10n emite warning inesperado;
- lockfile drift não é explicado;
- `shared_preferences`/backend muda sem revalidar NativeLocaleStore;
- manifest do baseline não corresponde à base upstream pretendida.

## Gate A — geração/toolchain

Usar a mesma versão de Flutter do upstream.

    flutter pub get
    flutter gen-l10n

A geração deve ser reproduzível. Se generated localization source for versionado:

    git diff --exit-code -- lib/l10n/generated

Falhar se imports/build dependem de generated code que não foi produzido ou se a política de versionamento diverge do plano.

## Gate B — ARB schema, parity e lifecycle

Validar:

- JSON válido;
- template key parity;
- `@metadata` obrigatório;
- placeholder names/types;
- ICU;
- plural/select;
- escaping;
- duplicate semantic keys;
- zero untranslated messages em shipping locale;
- zero key ARB órfã/não alcançada salvo allowlist justificada;
- zero allowlist stale;
- descrição semântica para novas keys;
- nenhuma key depende de texto de display como identidade.

## Gate C — ShippingLocales

Validar:

- shipping locales ⊆ generated locales;
- seletor expõe somente shipping;
- `en` permanece disponível;
- pseudo/debug locales nunca shipam;
- override persistido aceita somente `ProductLocaleId` shipping válido;
- `system` é sentinel separado;
- pt-BR só entra após promoção formal.

## Gate D — hardcoded/reachability/product text

Preferir scanner AST/token-aware em Dart e parsers adequados nas plataformas, complementados por regex.

Cobrir:

- `Text`, labels, titles, subtitles, hints;
- helper return strings e status/error functions;
- SnackBar/Dialog/tooltip/Semantics;
- Text.rich/RichText/TextSpan/InlineSpan;
- TextPainter/CustomPainter/Canvas;
- native setText/contentDescription/Toast/dialogs;
- Android XML string-bearing, menus, arrays, plurals e accessibility attrs;
- NotificationCompat, TaskNotification/background_downloader;
- Services/Receivers headless;
- PiP RemoteAction;
- FilePicker/plugin dialog titles;
- plist permission descriptions;
- macOS menu;
- Windows installer/registry descriptions;
- Linux `.desktop` e copy gerada em workflow;
- web shell/manifest/DOM;
- runtime JSON/YAML/CSV/Markdown;
- remote product copy/fallback assets;
- official help/setup/release content;
- links/QR oficiais acionados pelo app;
- SVG e runtime visual assets com texto;
- language display maps;
- TV keyboard submit/action labels;
- raw exception/provider/remote messages que alcançam UI;
- Clipboard/share/export/report/plugin/system human-text sinks.

Toda exclusão deve registrar ownership, reachability e motivo.

## Gate E — semantic coupling e presentation-cache identity

Falhar em novos casos de:

- branch/switch por display/localized text;
- persistência de localized string quando existe estado semântico;
- id/route/cache/database key derivados de label;
- protocolo cross-device usando mensagem humana como status/identity;
- UI escolhendo comportamento por exception message;
- cache singleton/static de copy localizada que pode sobreviver à troca de idioma sem `ProductLocaleId`/invalidation;
- output já localizado tratado como dado canônico.

## Gate F — formatters e localized human input

Testar en e pt-BR:

- date/time + timezone;
- 12h/24h quando a plataforma expõe preferência;
- number/percentage/rating/statistics;
- file size sem mudar silenciosamente a base 1024/1000;
- duration/relative time;
- plural;
- natural-language list formatting;
- parsing de número humano;
- round-trip input localizado → valor canônico → output localizado.

Separar explicitamente campos humanos de IP/URL/porta/PIN/ID/hash/schema tokens.

Rejeitar input ambíguo em vez de aplicar replace global de `,`/`.`.

## Gate G — Flutter analysis/test

    flutter analyze
    flutter test

Durante migração, suites segmentadas são permitidas. Antes da promoção, a suite suportada completa deve estar verde ou qualquer falha preexistente precisa de baseline reproduzível.

O harness comum deve fornecer delegates + ShippingLocales + locale explícito. Testes não destinados a wording não devem depender desnecessariamente de literal inglês.

## Gate H — Android native

Executar Gradle/Robolectric/device tests pertinentes.

Validar:

- parity `values` ↔ `values-pt-rBR` por nome e tipo;
- placeholders/formats/plurals/arrays;
- `AndroidTvTorrentPlayerActivity`;
- `TorboxTvPlayerActivity`;
- `MainActivity` PiP;
- `NativeLocaleBridge`;
- `NativeLocaleStore`;
- contract baseline `FlutterSharedPreferences / flutter.ui_locale_v1`;
- Dart write → native cold-read;
- corrupt/missing locale → fallback seguro;
- DevicePreferences/ProfilePreferencePortability;
- process-dead Services/Receivers;
- notification channels/actions;
- zero branch por display text.

## Gate I — platform build matrix

Compilar os targets publicados pelo upstream:

- Android;
- iOS;
- tvOS;
- macOS;
- Windows;
- Linux x64;
- Linux arm64.

Web deve compilar quando a superfície fizer parte do suporte esperado.

Mudança de matrix/workflow exige reclassificação no Gate Q.

## Gate J — pseudo/layout

Executar widget/golden/manual conforme superfície:

- pseudo-LTR expandido;
- pseudo-RTL;
- text scale;
- narrow phone;
- tablet/desktop;
- TV D-pad/focus;
- dialogs/bottom sheets/overlays/player;
- glyph/font coverage.

Não “corrigir” overflow reduzindo fonte agressivamente como regra geral.

## Gate K — secrets/log privacy

Mudanças de erro/localização não podem expor:

- tokens;
- credentials;
- signed URLs;
- payloads privados;
- filesystem paths sensíveis;
- secret-bearing exception detail.

UI localizada usa reason codes + argumentos seguros; diagnóstico bruto permanece em canal apropriado.

## Gate L — remote/runtime/official product copy

Bloquear quando:

- product-owned remote field não possui locale policy;
- shipping pt-BR cai para inglês em campanha/suporte oficial;
- catálogo remoto oficial não possui caminho pt-BR;
- cached remote copy pode servir locale anterior;
- runtime asset alcança UI sem inventory;
- release notes/Markdown oficial não têm ownership/policy;
- setup/help oficial essencial não tem locale policy;
- link oficial externo assume que browser `Accept-Language` == App language;
- QR em outro dispositivo é tratado como se herdasse o locale do app de origem.

Completeness report separa ARB, native resources, remote product copy, product-controlled catalog, runtime assets e official product content.

## Gate M — directionality e inline-text safety

Reportar app-owned runtime:

- `TextDirection.ltr/rtl` hardcoded;
- Alignment Left/Right;
- EdgeInsets/Positioned físicos quando start/end é semântico;
- directional icons/arrows;
- ordem rígida de spans para frase;
- dados LTR interpolados sem isolamento em contexto RTL.

Todo finding é corrigido ou allowlisted como físico/brand/technical com justificativa.

## Gate N — packaged artifact localization

Inspecionar artefato final, não apenas source:

- APK: `values-pt-rBR`, plurals e resources após shrink/minify;
- IPA/tvOS: `pt-BR.lproj`, InfoPlist.strings, target membership, TopShelf;
- macOS app/DMG: lproj + MainMenu;
- Windows installer: BrazilianPortuguese + CustomMessages/tasks/run;
- Linux AppImage x64/arm64: `Comment[pt_BR]` e source canônica;
- Web build: shell/manifest policy quando suportado.

Falhar se workflow/script sobrescreve uma fonte localizada por copy English-only.

## Gate O — real-device runtime localization smoke

Antes da promoção:

- Android phone/tablet: system en + app pt-BR e inverso;
- Android TV hardware: native players, D-pad/focus, TV keyboard, notifications/channels, PiP;
- tvOS hardware: focus/input/TopShelf;
- native shell smoke nas demais plataformas publicadas;
- screen reader em ao menos um cenário system locale != App language.

Registrar artifact SHA/build, device/OS, system locale, App language e evidência.

## Gate P — Unicode, composition e outbound text sinks

Falhar/allowlistar quando houver:

- natural-language `.join` com separador fixo;
- truncate/capitalize/initials por UTF-16 code unit;
- casing pós-localização não classificado;
- Clipboard/share/export/report/plugin/system human-text sink sem ownership;
- resultado localizado/composicionado persistido em vez de estado semântico;
- presentation cache não locale-keyed/invalidation-safe.

Testes obrigatórios:

- listas 0/1/2/3+;
- combining marks;
- non-BMP;
- ZWJ emoji;
- regional flag;
- en → pt-BR → en sem restart.

## Gate Q — upstream baseline drift e audit freshness

Fonte de verdade:

    docs/AUDIT_BASELINE_MANIFEST.json

Antes da implementação, rebases relevantes e promoção:

1. resolver a base upstream pretendida;
2. comparar com `auditedCommit`;
3. se mudou, obter diff completo;
4. classificar paths por reachability/ownership;
5. rerodar D–P no delta;
6. fazer full scan na promoção;
7. atualizar manifest somente após revisão;
8. validar tree SHA/contagens quando aplicável.

Falhar se:

- SHA mudou sem audit delta;
- novo target/path/asset/workflow não foi classificado;
- allowlist pertence a outra baseline;
- Flutter/dependency/plugin/backend relevante mudou sem revalidação;
- qualquer documento normativo usa IDs diferentes de `0,A..Q`.

## Release promotion contract

PT-BR só entra em `ShippingLocales` quando **0 + A..Q** aplicáveis estiverem verdes e a evidência mínima do plano estiver anexada. Não existe equivalência implícita entre IDs, e “não aplicável” precisa de justificativa versionada.
