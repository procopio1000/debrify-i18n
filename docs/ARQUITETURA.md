# Arquitetura i18n recomendada

## Stack

- Flutter `flutter_localizations`
- `gen_l10n`
- ARB
- ICU MessageFormat
- `intl`
- `AppLocaleController`
- `BuildContext.l10n`
- mappers localizados para enums/domínio
- recursos nativos por plataforma
- CI de completude

## Estado persistido

```text
ui_locale_v1
  system
  en
  pt-BR
```

`system` é o default. A preferência deve ser local ao dispositivo em V1.

## Separação de conceitos

```text
UI locale                 -> menus/controles/mensagens
metadata language         -> títulos/sinopses/artwork
audio preferred language  -> seleção de faixa
subtitle language         -> seleção de legenda
region                    -> availability/região
```

Nenhum desses conceitos deve alterar outro automaticamente.

## Fluxo

```text
SharedPreferences (device-local)
        |
        v
AppLocaleController
        |
        v
MaterialApp.locale
        |
        +--> AppLocalizations
        +--> Settings Search localized index
        +--> locale-aware formatters
        +--> NativeLocaleBridge -> Android TV native player
```

## Domínio

Models e services não devem receber `BuildContext` nem armazenar texto traduzido. Enums e valores canônicos são mapeados para mensagens localizadas na camada de apresentação.

## Android TV nativo

O override escolhido dentro do Flutter deve ser enviado explicitamente ao player/Activity nativo para evitar uma interface Flutter em PT-BR com player nativo em inglês.
