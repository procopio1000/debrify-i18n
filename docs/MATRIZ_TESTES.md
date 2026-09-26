# Matriz de testes i18n — V10

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
- LocaleFallbackPolicy independent of ShippingLocales
- future pt-PT addition does not silently mutate pt default
- preferred [pt-PT, en-US] resolves deterministically
- localeEpoch increments on effective locale change
- stale async completion cannot publish old-locale copy

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
| Declarative option registries | yes | yes | | yes | |
| Compact social metrics | yes | yes | | | |

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


## Remote/runtime product copy

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

## Rich text/custom painting

- Text.rich/RichText output is correct in en/pt-BR;
- styled/clickable placeholders can move position according to locale grammar;
- Semantics exposes the complete sentence;
- pseudo-RTL keeps spans, gestures and bidi isolation correct;
- launch idents/custom painters are localized or explicitly BRAND_ART_DIRECTION;
- no Canvas/TextPainter copy bypasses the inventory.

## Directionality

- app-owned hardcoded TextDirection is zero or explicitly classified;
- generic content text measurement uses ambient Directionality unless physically justified;
- semantic left/right paddings/alignments migrate to start/end;
- player timeline/media geometry intentionally physical remains stable;
- package/vendored LTR behavior carries ownership evidence rather than silent exclusion.


---

# Semantic/cross-device/runtime

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

# Composition/Unicode/outbound sinks

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


---

# Canonical locale, cache e baseline freshness

## ProductLocaleId

- `system` é sentinel separado de tags;
- `pt-BR` round-trip lógico;
- legado `pt_BR` -> `pt-BR`;
- `zh-Hant-TW` e `sr-Latn-RS` preservam language/script/region em harness não-shipping;
- `pt-PT` não cai silenciosamente em pt-BR;
- `en-US-u-hc-h12` não é persistido após descartar extension;
- `ca-ES-valencia` não é persistido após descartar variant;
- private-use/malformed tag -> fallback seguro sem regravação truncada;
- representation adapters produzem ARB/Android/Apple/Linux forms esperadas.

## Android physical locale contract

Baseline:

    FlutterSharedPreferences
    flutter.ui_locale_v1

Testar:

- Dart `DevicePreferences.setString('ui_locale_v1', 'pt-BR')` -> native cold-read pt-BR;
- `system` -> contexto nativo resolve pela lista do SO;
- ausência -> system/en seguro;
- valor inválido/corrompido -> fallback seguro;
- process kill antes de Service/Receiver;
- upgrade de plugin/backend invalida contract fixture e bloqueia Gate 0/Q;
- nenhum mirror `ui_locale_native_*`.

## Presentation cache locale flip

Sem restart:

1. en;
2. abrir Settings Search/overlay/dialog relevante;
3. trocar para pt-BR;
4. provar que copy long-lived não continua en;
5. voltar para en;
6. repetir com remote cached copy quando aplicável.

Cobrir:

- `LocalizedCopyResolver`;
- Settings Search index;
- remote product copy cache;
- menus/overlays que mantêm estado;
- native actions/channels quando plataforma permite atualização;
- headless notification disparada após a troca.

## Official external content

- App language pt-BR + browser/system en: link oficial passa locale explícito quando contrato do destino suporta;
- App language en + browser/system pt-BR: mesmo princípio;
- URL sem locale explícito possui landing/selector/fallback documentado;
- QR fixo aberto em outro dispositivo não é assertado como “seguindo o app de origem”;
- WebDAV setup continua funcional independentemente da política de locale.

## Gate Q / baseline drift

- manifest SHA == upstream base -> fast path verde;
- upstream base diferente -> diff obrigatório;
- path Dart novo user-facing -> D–P reexecutados;
- novo Android resource/Activity/Service -> H/N/O conforme reachability;
- novo asset/config/workflow de packaging -> D/L/N;
- mudança Flutter/`shared_preferences` -> Gate 0 + contratos dependentes;
- tree recursive incompleta/truncated -> falha até estratégia completa;
- allowlist de baseline antigo -> rejeitada;
- documentos com gate alias/renumeração -> falha.

## Gate registry consistency

Assertar em CI/document lint:

- plano contém exatamente IDs `0,A..Q`;
- `CI_QUALITY_GATES.md` usa os mesmos IDs;
- matriz referencia IDs canônicos;
- nenhum documento normativo cria `Gate 1..14` ou `Gate 12 / Gate L`;
- histórico de versão fica em `AUDITORIA_V*.md`, não como contrato paralelo.


## Accessibility locale attribution

- system en + App language pt-BR: subtree de copy própria expõe pt-BR no locale semântico esperado;
- system pt-BR + App language en: subtree expõe en;
- troca en → pt-BR sem restart atualiza também language attribution;
- external/user data em outro idioma não é recategorizado cegamente;
- screen reader real em ao menos um target confirma comportamento e registra limitações da voz/plataforma.


---

# V9 — hardening temporal, concorrência e continuidade

## Calendar/date presentation

- Trakt calendar full date/month/headline/short weekday en/pt-BR;
- IPTV EPG Today/Tomorrow/Yesterday en/pt-BR;
- recordings human date en/pt-BR;
- zero English manual month/weekday table em shipping presentation fora de allowlist;
- first visual day-of-week pode seguir locale sem alterar provider fetch semantics.

## Time taxonomy

- CIVIL_TIME respeita preferência 12/24h salvo PRODUCT_FIXED_CLOCK justificado;
- MEDIA_TIMECODE permanece mm:ss/h:mm:ss;
- PROTOCOL_DATE_TIME/FILENAME_TIMESTAMP/DIAGNOSTIC_TIMESTAMP não mudam com App language;
- PROVIDER_CALENDAR_RULE não muda com UI locale.

## Model/presentation boundary

- expiration/date/size saem de domain como dados tipados;
- absence não usa English `N/A` como contrato de domínio;
- presentation aplica locale somente a valores humanos;
- IDs/PIN/porta/S01E02/timecode permanecem invariantes.

## Async locale epoch

- en A starts -> pt-BR -> B completes -> A completes late -> UI continua pt-BR;
- repetir pt-BR -> en;
- semantic payload neutro pode sobreviver; copy materializada não;
- Settings Search, remote config e LocalizedCopyResolver usam o mesmo snapshot contract.

## Search Unicode

- composed/decomposed `Configuração`;
- `áudio/audio`, `conexão/conexao`, `reprodução/reproducao`;
- combining marks/emoji não quebram normalização;
- normalização altera somente search key.

## Collation

- ordem de produto/provider permanece estável quando não há requisito alfabético;
- search-fold não é usado como collator;
- collator real, se introduzido, recebe testes en/pt-BR e cross-platform.

## Android authority

- pt-BR shipping funciona sem segunda autoridade Android per-app language;
- source guard detecta adoção de localeConfig/LocaleManager/AppCompat sem migration contract;
- cold-start/background continua lendo `ui_locale_v1`.

## Mixed-language semantics

- pt-BR app chrome + external title inglês;
- en app chrome + filename/user data português;
- app-owned semantics recebe App language;
- idioma externo desconhecido não é inventado.

## Locale-switch continuity

- navigation stack preservado;
- form/query não salvo preservado;
- D-pad focus preservado/restaurado semanticamente;
- playback source/position/session preservado;
- download/recording preservado;
- remote pairing/session preservado;
- somente presentation/index caches são invalidados.

## Runtime packages

- scan recursivo de `packages/**` runtime roots;
- classificação FIRST_PARTY_FORK/VENDORED_THIRD_PARTY/GENERATED;
- app-owned patch copy em package vendorizado não é silenciosamente excluída;
- artifact inspection confirma recursos localizados quando package muda.


---

# V10 — indirect presentation, temporal e coverage closure

## Declarative options/registries

- `ContentDisplayMatchMode`: storageKey/behavior invariantes em en/pt-BR; label localizado;
- `AndroidVideoRendererMode`: storageKey/videoOutput/hardwareDecoder invariantes; label/description localizados;
- external players: executable/bundle/brand invariantes; System Default/Custom App/Custom Command/descriptions localizados;
- `MetadataCategory` e `PlaylistViewMode`: identidade/behavior invariantes e display localizado;
- Settings Search consome o mesmo mapper das telas;
- locale flip atualiza labels sem regravar preferência;
- launch-ident brand vs PRODUCT_COPY é classificado e testado.

## Temporal presentation

- `CalendarTimeFormat` não possui label app-owned fixo;
- device mode respeita preferência do dispositivo;
- explicit 12h/24h não concatena AM/PM manual;
- `StremioTvNowPlaying` expõe estado/datas, não frase final;
- tuner localiza Ended/Ends at usando o locale efetivo;
- Debrify TV StatsTile formata `lastSearchedAt` como CIVIL_TIME;
- TV time picker localiza título, instrução e ações mantendo AM/PM via MaterialLocalizations e D-pad intacto.

## Compact metrics

- YouTube views, Reddit score e Lemmy score mantêm números canônicos;
- 999 / 1.000 / 1.250 / 999.999 / 1.000.000;
- en / pt-BR;
- palavra app-owned `views` não vive no service/model;
- sort/filter/API não consome string compactada.

## Coverage closure

- todo path no `RuntimeSurfaceUniverse` aparece exatamente uma vez como reachable/classified ou excluded-with-evidence;
- path novo quebra Gate Q;
- blob alterado em source sensível exige re-scan;
- generated registra source generator;
- vendored patch com app-owned copy recebe ownership apropriado;
- manifest stale falha CI.
