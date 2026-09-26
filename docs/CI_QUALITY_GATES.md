# Quality Gates i18n — V7

## Gate 0 — dependency compatibility

Baseline:

    Flutter 3.44.8
    flutter_localizations -> intl 0.20.2

Run:

    flutter --version
    flutter pub get
    flutter pub deps
    flutter gen-l10n

Fail if:

- intl does not resolve 0.20.2 under the pinned baseline;
- dependency_overrides masks intl;
- l10n.yaml contains synthetic-package;
- gen-l10n emits an unexpected warning;
- lockfile drift is unexplained.

## Gate 1 — pinned toolchain

Use the same Flutter version as upstream CI.

    flutter pub get
    flutter gen-l10n

Generation must be reproducible. When generated localization source is committed, run `git diff --exit-code -- lib/l10n/generated` after generation.

## Gate 2 — ARB schema and parity

Validate:

- valid JSON;
- template key parity;
- @metadata required;
- placeholder name/type parity;
- ICU syntax;
- plural/select validity;
- escaping;
- zero untranslated messages for shipping locales.

## Gate 3 — ShippingLocales

Validate:

- every shipping locale exists in generated locales;
- selector exposes only shipping locales;
- locale resolver accepts only shipping overrides;
- development/pseudo locales never ship accidentally.

## Gate 4 — hardcoded user-facing text

    dart run tool/l10n_audit.dart

Scan beyond Text(...):

- titles/labels/subtitles;
- helper return strings;
- status/error functions;
- SnackBar/Dialog;
- tooltip/hint;
- Semantics;
- native setText/contentDescription/Toast/dialogs;
- Android XML android:text/contentDescription/hint;
- NotificationCompat title/text/action/channel;
- Services/Receivers that can run without Flutter;
- macOS native menu copy;
- Windows installer copy;
- plist permission descriptions;
- web shell metadata and DOM lang/dir;
- Text.rich/RichText/TextSpan/InlineSpan;
- TextPainter/CustomPainter/Canvas text;
- TaskNotification/background_downloader copy;
- runtime-loaded JSON/YAML/CSV/Markdown que alimenta UI;
- remote product copy e fallback assets;
- MarkdownBody/Markdown sources;
- official setup/help links and QR destinations;
- SVG text and runtime visual assets with potential embedded copy.

All exceptions require a versioned reason.

## Gate 5 — semantic coupling

    dart run tool/l10n_semantic_coupling_audit.dart

Fail on new patterns where display/localized text is used as:

- branch condition;
- switch discriminator;
- persisted value;
- route/id;
- cache/database key.

## Gate 6 — platform resources

Compare Android default and pt-BR resource keys.

Audit Apple InfoPlist.strings/project localizations.

Audit tvOS Top Shelf resources.

Audit Web manifest/index and runtime lang/dir.

Audit Windows installer language/custom messages and Runner.rc metadata.

Audit macOS MainMenu localization.

Audit NativeLocaleStore/background notification resources.

Validate locale storage architecture:

- ui_locale_v1 is registered in DevicePreferences.allowedKeys;
- ProfilePreferencePortability rejects ui_locale_v1;
- no ui_locale_native_mirror_* key exists;
- no new raw SharedPreferences access is introduced by i18n;
- Android backup rules continue excluding SharedPreferences unless a deliberate policy change is reviewed.

## Gate 7 — formatters

Test en and pt_BR for:

- date;
- time;
- number;
- decimal file size;
- percentage;
- duration;
- relative time.

## Gate 8 — analysis

    flutter analyze

## Gate 9 — tests

Run targeted l10n tests during each phase.

Before PT-BR promotion run the full supported upstream suite or document a reproducible pre-existing baseline for unrelated failures.

## Gate 10 — Android native tests

Run relevant Gradle/Robolectric tests for:

- AndroidTvTorrentPlayerActivity;
- TorboxTvPlayerActivity;
- NativeLocaleBridge;
- NativeLocaleStore single-store contract;
- DevicePreferences/ProfilePreferencePortability contract;
- resource resolution;
- process-dead Service/Receiver notification;
- existing notification channel after locale change;
- notification actions/plurals;
- zero branch based on localized/display text.

## Gate 11 — layout/a11y

Test:

- pseudo-LTR expansion;
- pseudo-RTL;
- text scale;
- narrow phone;
- desktop;
- TV D-pad/focus;
- Semantics/contentDescription.

## Gate 12 / Gate L — remote/runtime product copy

Fail or block promotion when:

- product-owned remote field is user-facing but has no locale policy;
- shipping locale falls back to English for official campaign/support copy;
- official engine-catalog editorial copy has no pt-BR path;
- cached remote copy can retain the previous locale variant;
- a runtime asset reaches UI without inventory/classification;
- official Markdown/release content has no ownership/locale policy;
- an essential official setup/help destination has no documented locale policy;
- fixed Settings copy still comes from English remote config instead of ARB.

Completeness report must expose ARB, native resources, remote product copy, product-controlled remote catalog and runtime asset copy separately.

## Gate 13 / Gate M — directionality and inline-text safety

Report app-owned runtime occurrences of:

- hardcoded TextDirection.ltr/rtl;
- Alignment left/right;
- semantic EdgeInsets/Positioned using physical left/right;
- directional icons;
- TextSpan fragment ordering that assumes English grammar.

Every finding must be fixed or allowlisted with physical/brand/third-party justification. Pseudo-RTL must cover rich text and mixed-direction data.

## Gate 14 — release

Do not add PT-BR to ShippingLocales until:

- ARB completeness = 100%;
- native resources = 100%;
- semantic coupling findings = 0;
- unapproved hardcoded UI findings = 0;
- platform and critical runtime tests pass;
- Windows installer pt-BR passes;
- macOS menu pt-BR passes;
- Web lang/dir passes;
- background Android locale after process death passes;
- stale allowlist entries = 0;
- remote product copy completeness = 100% for shipping pt-BR;
- runtime asset findings unclassified = 0;
- hardcoded directionality findings unclassified = 0;
- rich-text/custom-painter findings unclassified = 0.


---

# Adições V6

## Extensões dos gates existentes

**Gate B:** detectar keys ARB órfãs/não alcançadas e allowlist stale.  
**Gate D:** incluir language display maps, TvTextField/TvKeyboard action labels e raw error/remote messages em sinks de UI.  
**Gate E:** proibir display text/localized text como result identity em protocolo cross-device e branches por exception message.  
**Gate F:** testar parsing de input humano pt-BR/en e separar tokens técnicos invariantes.

## Gate O — Real-device runtime localization smoke

Antes de promover pt-BR:

- Android phone/tablet: system en ↔ app pt-BR e system pt-BR ↔ app en;
- Android TV hardware: native players, D-pad/focus, keyboard, notifications/channels, PiP quando suportado;
- tvOS hardware: focus/input/Top Shelf e boundary de system/app language;
- native shell smoke nas demais plataformas publicadas;
- accessibility/screen reader em ao menos um cenário system locale diferente do App language.

Toda evidência registra artifact SHA/build, device/OS, system locale, App language e resultado.


---

# Gate P — Unicode, composition e outbound text sinks

Falhar ou exigir allowlist versionada quando um path user-facing possuir:

- lista natural criada por `.join(', ')`/separador fixo sem formatter/ICU/classificação;
- helper visual de truncate/capitalize/initials baseado em `String.length`, `substring`, `s[0]` ou code unit;
- casing pós-localização não classificado;
- Clipboard/share/export/report/plugin/system human-text sink sem ownership;
- localized string persistida como resultado de composição quando dados semânticos poderiam ser persistidos.

Exigir testes para 0/1/2/3+ listas e para combining mark, non-BMP, ZWJ emoji e regional flag nos helpers grapheme-sensitive.
