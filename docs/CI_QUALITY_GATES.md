# Quality Gates i18n

## Gate 1 — geração

```bash
flutter pub get
flutter gen-l10n
```

## Gate 2 — ARB

Validar:

- JSON válido;
- todas as keys do template existem em todos os locales shipping;
- metadata `@key` obrigatória;
- placeholders compatíveis;
- ICU válido;
- zero mensagens não traduzidas em locale shipping.

## Gate 3 — hardcoded UI

```bash
dart run tool/l10n_audit.dart
```

Deve falhar quando surgir nova string user-facing não allowlisted.

## Gate 4 — análise

```bash
flutter analyze
```

## Gate 5 — testes

```bash
flutter test test/l10n
```

Depois ampliar para as suites impactadas por cada fase.

## Gate 6 — Android resources

Comparar keys de:

```text
values/strings.xml
values-pt-rBR/strings.xml
```

Marcas e tokens técnicos podem ser allowlisted.

## Gate 7 — release

Não declarar um locale em `supportedLocales` até atingir 100% de cobertura e passar os testes de layout/placeholder.
