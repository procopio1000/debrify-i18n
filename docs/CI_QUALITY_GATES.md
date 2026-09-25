# Quality Gates i18n — V2

## Gate 1 — pinned toolchain

Use the same Flutter version as upstream CI.

    flutter pub get
    flutter gen-l10n

Generation must be reproducible.

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
- plist permission descriptions;
- web shell metadata.

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

Audit Web manifest/index.

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
- locale bridge;
- resource resolution.

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
- platform and critical runtime tests pass.
