# Quality Gates i18n — V3

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
- web shell metadata and DOM lang/dir.

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

## Gate 12 — release

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
- stale allowlist entries = 0.
