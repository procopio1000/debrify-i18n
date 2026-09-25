# Matriz de testes i18n — V2

## Locales

### Shipping inicial

- en

### Primeiro locale a promover

- pt_BR

### Debug/test only

- pseudo-LTR expandido
- pseudo-RTL

## Controller e resolução

- fresh install
- system en
- system pt-BR
- preferred locale list
- manual en
- manual pt-BR
- invalid persisted locale
- legacy pt_BR normalization
- unknown locale
- system locale change while following system
- system locale change with manual override
- restart persistence

## Isolation invariants

- UI locale does not alter metadata language
- UI locale does not alter artwork language
- UI locale does not alter audio preference
- UI locale does not alter subtitle preference
- UI locale does not alter region
- UI locale does not alter filter/sort ids
- profile switch does not alter UI locale
- WebDAV does not alter UI locale
- profile restore/import does not alter UI locale

## Surface matrix

| Surface | Unit | Widget | Golden/Pseudo | Manual TV | Native |
|---|---:|---:|---:|---:|---:|
| App root/bootstrap | yes | yes | yes | yes | |
| Startup failure/recovery | yes | yes | yes | yes | |
| Profiles | yes | yes | yes | yes | |
| Onboarding | yes | yes | yes | yes | |
| SettingsRows registry | yes | yes | | yes | |
| Settings Search | yes | yes | yes | yes | |
| Home/Discover/Search | | yes | yes | yes | |
| Details/Collections/See All | | yes | yes | yes | |
| Sources/Addons/Filters | yes | yes | yes | yes | |
| Debrid/Cloud/Downloads | yes | yes | yes | yes | |
| Flutter Player | yes | yes | yes | yes | |
| AndroidTvTorrentPlayerActivity | yes | | | yes | yes |
| TorboxTvPlayerActivity | yes | | | yes | yes |
| IPTV/Debrify TV/Stremio TV | yes | yes | yes | yes | |
| Tracking/Calendar | yes | yes | | yes | |
| Sync/Backup/Remote | yes | yes | | yes | |
| Formatters | yes | | | | |
| iOS/tvOS/macOS native copy | yes | | | manual | yes |
| Web shell metadata | yes | | | | yes |
| Windows/Linux runner copy | yes | | | | yes |

## Settings Search

Test:

- localized titles/subtitles/categories
- localized keywords
- configuracoes matches Configurações
- whitespace normalization
- locale change rebuilds index
- blank query shows full index
- D-pad moves from field to first result
- toggles remain live

## Semantic-coupling regression

Tests/static checks must prove:

- display labels are not persistence values
- All sources logic uses stable id
- localized enum label is never branch identity
- locale change cannot alter protocol output

## Android native locale matrix

Test both native player Activities:

1. system en + system mode
2. system pt-BR + system mode
3. system pt-BR + override en
4. system en + override pt-BR

Verify resources, focus, dialogs, overlays and no unintended playback restart.

## Layout/a11y

- 30–40% pseudo expansion
- RTL directionality
- mixed LTR/RTL external data
- 200% text scaling where platform allows
- narrow phone
- 720p TV
- 1080p TV
- desktop window
- Semantics
- Android contentDescription
- font glyphs for PT-BR accents

## Platform-native

### Android

- key parity values vs values-pt-rBR
- plurals
- contentDescription
- notifications/dialogs/Toast
- NativeLocaleBridge

### Apple

- InfoPlist.strings
- NSLocalNetworkUsageDescription
- Xcode supported languages
- tvOS Runner
- Top Shelf extension
- macOS native copy

### Web

- no A new Flutter project placeholder
- canonical name/description
- title/manifest consistency

## Release regression

Before promoting pt-BR:

- generated localization is reproducible
- ARB completeness 100%
- platform resource parity 100%
- hardcoded UI findings 0 except approved allowlist
- semantic coupling findings 0
- critical Flutter tests pass
- Android native tests pass
- platform build matrix passes
- PT-BR human in-context review completed
