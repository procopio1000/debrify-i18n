# Matriz de testes i18n — V7

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
- system pt-PT (must not auto-map to pt-BR)
- system pt without region
- en-US / en-GB -> en
- preferred locale list
- manual en
- manual pt-BR
- invalid persisted locale
- legacy pt_BR normalization
- unknown locale
- system locale change while following system
- system locale change with manual override
- restart persistence
- BCP47 persisted pt-BR vs gen_l10n pt_BR
- corrupt/unknown canonical native locale backing preference
- App language vs OS-owned localization boundary

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
| Android background notifications | yes | | | yes | yes |
| Non-Android plugin notifications | yes | yes | | manual | yes |
| Remote product copy | yes | yes | | | yes |
| Runtime asset copy | yes | yes | | | yes |
| Rich text/custom painting | yes | yes | yes | yes | |
| Markdown/release notes | yes | yes | yes | | |
| Official WebDAV setup guide/QR | yes | yes | | yes | |
| macOS MainMenu | yes | | | manual | yes |
| Windows installer | yes | | | manual | yes |
| Web DOM lang/dir | yes | yes | | | yes |
| TV voice/input locale | yes | yes | | yes | yes |
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

### Android cold-start/background

With Flutter process killed:

1. scheduled recording alarm posts localized failure/success UI;
2. download/recording foreground service uses NativeLocaleStore;
3. action labels Pause/Resume/Cancel/Stop are localized;
4. 0/1/2 summary plurals are correct;
5. existing notification channel gets current localized name/description where Android permits;
6. corrupt/unknown ui_locale_v1 falls back safely;
7. NativeLocaleStore reads the same canonical preference and no mirror key exists;
8. no state decision depends on translated text.

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


## Voice/input locale

- system-default voice remains unchanged when only App language changes
- explicit BCP-47 reaches Android recognizer only when input policy requests it
- PT-BR TV user can enter accented Portuguese characters through a supported path
- recognizer unavailable/permission denied states are localized
- transcript remains user/external data and is never translated

## Device preference / portability

- ui_locale_v1 is accepted by DevicePreferences
- raw SharedPreferences access count/source guard does not regress
- ProfilePreferencePortability rejects ui_locale_v1
- profile switch leaves app language unchanged
- WebDAV/portable profile backup does not contain ui_locale_v1
- Android cloud/device transfer remains excluded by current backup rules
- app/device reset leaves no orphan native locale state
- tvOS shared_preferences_tvos can read/write the small device key safely

## Artifact reachability

- runtime local packages are scanned
- dev/test/example/generated code is classified rather than silently excluded
- SYSTEM_OWNED_UI and THIRD_PARTY_OWNED_UI findings carry ownership evidence
- stale allowlist entry fails CI


## V4 remote/runtime copy

- SupportRemoteConfig fixed Settings labels come from ARB;
- fallback asset and cached config resolve correctly in en and pt-BR;
- changing App language re-resolves campaign copy without stale cache language;
- malformed/partial remote payload falls back safely;
- shipping pt-BR reports English fallback of official copy as incomplete;
- official engine catalogue has a pt-BR editorial path;
- third-party imported engine copy remains external data;
- runtime JSON/YAML/CSV/Markdown reaching UI is inventoried;
- offline mode preserves a deterministic locale fallback;
- release.body has explicit OFFICIAL_PRODUCT_CONTENT ownership;
- fixed release-note fallback chrome is localized;
- WebDAV setup guide link/QR destination has a documented locale policy;
- one stable QR can reach a locale-negotiating landing page or equivalent;
- visual runtime assets with potential text are reviewed/classified.

## V4 rich text/custom painting

- Text.rich/RichText output is correct in en/pt-BR;
- styled/clickable placeholders can move position according to locale grammar;
- Semantics exposes the complete sentence;
- pseudo-RTL keeps spans, gestures and bidi isolation correct;
- launch idents/custom painters are localized or explicitly BRAND_ART_DIRECTION;
- no Canvas/TextPainter copy bypasses the inventory.

## V4 directionality

- app-owned hardcoded TextDirection is zero or explicitly classified;
- generic content text measurement uses ambient Directionality unless physically justified;
- semantic left/right paddings/alignments migrate to start/end;
- player timeline/media geometry intentionally physical remains stable;
- package/vendored LTR behavior carries ownership evidence rather than silent exclusion.


---

# Matriz V6 — semantic/cross-device/runtime

## Language display names
- eng/spa/por/pt-BR/por-br: identidade e display en/pt-BR;
- unknown code fallback técnico;
- paridade Dart/Android;
- external provider labels permanecem dados.

## Localized numeric input
- pt-BR: 8,5;
- en: 8.5;
- persistência canônica;
- inválidos/ambíguos;
- IP/URL/porta/PIN não localizados.

## Error contract
- reason code -> AppLocalizations;
- PlatformException sem depender de message;
- raw provider detail classificado/redacted.

## Remote multi-device
- sender en -> receiver pt-BR;
- sender pt-BR -> receiver en;
- success/failure/admin/retry/pairing;
- backward compatibility com payload legado.

## TV keyboard
- default submit actions;
- custom keyboardSubmitLabel;
- Clear/paste/voice/backspace semantics;
- OverlayEntry mantém locale.

## Gate O
- artifact/device/system locale/App language/evidência por cenário real.


---

# Matriz V7 — composition/Unicode/outbound sinks

## List composition

- 0/1/2/3+ itens em en e pt-BR;
- services/trackers de onboarding mantêm brands e localizam somente gramática;
- failure lists não concatenam frase inglesa + join fixo;
- middle-dot metadata é classificada como VISUAL_METADATA_LIST;
- multiline Remote list mantém labels como data e localiza chrome/contagem.

## Grapheme safety

- truncation de folder/profile/external display name não corta `e\u0301`;
- emoji non-BMP permanece inteiro;
- família/ZWJ permanece inteira;
- regional-flag pair permanece inteiro;
- initials/capitalize não indexam primeiro code unit como “caractere”.

## Casing

- tradução não recebe uppercase/lowercase pós-resolução salvo caso allowlisted;
- technical badges continuam estáveis;
- third-party/user labels preservam conteúdo salvo requisito explícito.

## Outbound sinks

- Clipboard de URL/PIN/token é TECHNICAL/USER_DATA e não traduz payload;
- qualquer copy humana copiada/compartilhada/exportada usa locale efetivo;
- FilePicker/plugin/system title fornecido pelo app continua coberto como APP_SUPPLIED_SYSTEM_UI;
- reports legíveis distinguem product copy de diagnostic/external detail.
