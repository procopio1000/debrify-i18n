# Auditoria Baseline — Debrify v0.10.0-beta.1

## Referência

- Upstream: `varunsalian/debrify`
- Tag: `v0.10.0-beta.1`
- Commit: `9619c10b06ee919cacbe996b30be7739dc09c6d6`
- Auditoria: 2026-09-25

## Principais constatações

1. `intl` existe no `pubspec.yaml`.
2. Não existe estrutura Flutter de localização da UI.
3. Não foram encontrados `flutter_localizations`, `supportedLocales`, `localizationsDelegates`, `AppLocalizations` ou ARB de UI.
4. A árvore auditada contém aproximadamente 3.666 entradas, 1.757 arquivos Dart, 508 arquivos Dart diretamente em `lib/screens`, `lib/widgets` e `lib/theme`, 614 testes Dart e 743 arquivos de plataforma.
5. A busca estática encontrou ampla superfície user-facing: ~170 arquivos com `Text('...')`, ~281 com `title:`, ~326 com `label:`, ~147 com `subtitle:`, ~56 com `tooltip:`, ~55 com `hintText:`, ~115 com `SnackBar(`, ~77 com `AlertDialog(`, ~32 com `Semantics(` e ~152 com `this.label`.
6. `lib/main.dart` possui múltiplos `MaterialApp` para app principal e fluxos de bootstrap/recovery.
7. Android nativo já possui `strings.xml`, mas ainda há strings visíveis hardcoded em Kotlin.
8. `lib/utils/formatters.dart` usa padrões fixos de data em inglês e formatação decimal manual.
9. Há ampla utilização de `toUpperCase()`, que deve ser revisada em textos user-facing para internacionalização futura.
10. Idioma da UI deve permanecer separado de metadados, áudio, legendas e região.

## Áreas de maior risco

- `lib/main.dart`
- `lib/screens/search_screen.dart`
- `lib/services/storage_service.dart`
- `lib/screens/video_player_screen.dart`
- `lib/screens/magic_tv_screen.dart`
- `lib/widgets/initial_setup_flow.dart`
- Android TV native player
- Settings Search
- models/enums com `.label`
- recovery/bootstrap
- WebDAV/backup/remote transfer
