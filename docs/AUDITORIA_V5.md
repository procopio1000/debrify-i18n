# Auditoria V5 — Debrify i18n/l10n

**Data:** 2026-09-25
**Plano resultante:** `PLANO_MESTRE_V5.md`
**Upstream:** `varunsalian/debrify`
**Release verificada:** `v0.10.0-beta.1`
**Commit auditado:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`
**Git tree auditada:** `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`

## Método

A auditoria V5 revalidou o plano V4 contra o código e o packaging reais da baseline, com foco em rotas de escape que uma busca convencional por `Text(...)` não cobre:

- árvore Git completa e roots de produto;
- Flutter/Dart e test harnesses;
- Android Kotlin/XML, Activities, PiP, Services/Receivers e accessibility;
- iOS/tvOS/macOS/Windows/Linux/Web;
- workflow de release e conteúdo gerado durante packaging;
- app-supplied system UI;
- runtime assets/remote copy já cobertos pela V4;
- testes e evidência pós-build dentro dos artifacts.

## Baseline estrutural reproduzível

A árvore exata possui **3.199 blobs/arquivos**.

Contagem por roots first-party de produto:

- `lib/`: 972
- `android/`: 530
- `assets/`: 53
- `ios/`: 49
- `linux/`: 12
- `macos/`: 32
- `tvos/`: 62
- `web/`: 7
- `windows/`: 19
- total desses roots: **1.736**
- incluindo `packages/` (373): **2.109**

Esses números não significam que 2.109 arquivos possuam copy user-facing; são uma base estrutural determinística para o scanner de reachability.

## Achados novos V5

### 1. Linux AppImage sobrescreve a fonte `.desktop`

`linux/debrify.desktop` possui:

    Comment=Debrid client for torrents and streaming

Porém `.github/workflows/build.yml` cria **outro** `AppDir/debrify.desktop` inline nos jobs x64 e arm64 com:

    Comment=Modern debrid companion with torrent search and playlist management

Isso ocorre aproximadamente nas regiões 680–705 e 883–907 da baseline auditada. Portanto, localizar apenas `linux/debrify.desktop` deixaria o AppImage publicado em inglês.

**Correção no V5:** uma única fonte canônica `.desktop`, `Comment[pt_BR]`, scan de workflow/scripts e Gate N que inspeciona ambos os AppImages finais.

### 2. PiP Android contém copy hardcoded renderizada pelo SO

`android/app/src/main/kotlin/com/debrify/app/MainActivity.kt` contém em `buildPipActions()`:

    val title = if (pipIsPlaying) "Pause" else "Play"
    makePipAction(..., "Next", ...)

e usa o mesmo `title` como title e content description da `RemoteAction`.

**Correção no V5:** classificar como `APP_SUPPLIED_SYSTEM_UI`, mover para resources, resolver pelo Context localizado e testar atualização de PiP ativo em en/PT-BR.

### 3. FilePicker é outro exemplo de app-supplied system UI

Exemplos confirmados de `dialogTitle`:

- `lib/screens/settings_screen.dart` — `Save diagnostic logs`, `Choose Debrify backup file`;
- `lib/screens/profiles/profile_recovery_screen.dart` — `Choose a Debrify backup`;
- `lib/widgets/remote/remote_control_screen.dart` — `Choose a profile avatar`;
- `lib/screens/profiles/edit_profile_screen.dart` e `self_profile_settings_page.dart` — `Choose an avatar image or GIF`;
- `lib/screens/settings/collections_settings_page.dart` — `Export collection`;
- `lib/screens/settings/external_player_settings_page.dart` — `Select Video Player Application`.

O SO/plugin pode desenhar a janela, mas esses títulos são fornecidos pelo Debrify.

**Correção no V5:** `APP_SUPPLIED_SYSTEM_UI`; separar copy controlada pelo app do chrome realmente pertencente ao SO.

### 4. XML Android precisa de cobertura mais ampla

A baseline possui literals como:

- `view_subtitle_settings_panel.xml` — `SUBTITLE SETTINGS` e ajuda de navegação;
- `view_stremio_tv_guide.xml` — `Search channels...`;
- `view_unified_channel_guide.xml` — `Search or type channel number...`;
- `view_iptv_channel_guide.xml` — `Search this source — press OK`, `SAVED`, `Channels`;
- `activity_torbox_tv_player.xml` — `SWITCHING CHANNEL`;
- `activity_android_tv_torrent_player.xml` — `Back`;
- diversos `contentDescription`, estados, badges e símbolos.

Símbolos como ★/✓/setas podem ser iconografia/technical e precisam de classificação, não tradução automática.

**Correção no V5:** scan de todos os XMLs runtime e resource types/attributes string-bearing, incluindo arrays, menus, plurals, accessibility e review de `translatable=false`.

### 5. Resource parity precisa validar formato/placeholder

Comparar apenas nomes de resources é insuficiente. O V5 exige paridade de tipo e de placeholders `%1$s`, `%1$d`, argumentos posicionais, plurals e flags de formatação entre default e `values-pt-rBR`.

### 6. Faltava um materializador de localização sem BuildContext

A V4 já proibia injetar `BuildContext` em services e identificava `TaskNotification` do `background_downloader`, mas não definia um mecanismo comum para obter `AppLocalizations` fora da árvore de widgets.

**Correção no V5:** `LocalizedCopyResolver` (nome conceitual), recebendo o locale efetivo como entrada e usando o delegate gerado. Ele não persiste nem decide locale e, portanto, não se torna uma segunda autoridade.

### 7. Framework localization delegates precisam ser contrato explícito

A baseline usa `showDatePicker`/`showTimePicker` em `lib/screens/settings/recordings_page.dart`. O V5 torna obrigatório que todo root relevante use os delegates do app + Material + Widgets + Cupertino, preferencialmente `AppLocalizations.localizationsDelegates`.

### 8. Test harness é parte da migração

A busca indexada retornou o limite de 100 ocorrências de `MaterialApp(`; **93** delas estavam em `test/`. Portanto, existem pelo menos 93 arquivos de teste com MaterialApp próprio na amostra retornada. Também há assertions por wording (`find.text`, `find.byTooltip`) em inglês.

**Correção no V5:** `localizedTestApp`/helper comum, delegates completos, locale explícito e desacoplamento de wording em behavior tests.

### 9. Accessibility nativa mistura dado externo e copy

`android/app/src/main/kotlin/com/debrify/app/tv/TvSourceBrowserController.kt`, linha 557 da baseline:

    contentDescription = "${entry.title}, Playing"

`entry.title` continua external data; `Playing` é copy localizável.

**Correção no V5:** formatted resource com placeholder + bidi safety.

### 10. Web DOM e PWA manifest têm autoridades diferentes

`web/manifest.json` e `web/index.html` ainda possuem metadata/template estática na baseline. O DOM pode reagir ao App language; metadata de instalação/cache do PWA não deve ser tratada como se fosse live runtime state.

**Correção no V5:** distinguir DOM runtime de manifest/install metadata e escolher policy neutra ou manifest/HTML locale-aware servido pelo host.

### 11. Windows protocol descriptions precisam de classificação

`windows/installer.iss` grava descrições humanas como `URL:Stremio Protocol`, `URL:Magnet Protocol` e `URL:Debrify Protocol`. Os protocol identifiers/registry keys são técnicos e nunca devem ser traduzidos; a descrição humana é auditada separadamente.

### 12. Source-level green não prova artifact-level localization

O workflow atual publica Android APK, macOS DMG, Windows installer, iOS IPA, tvOS IPA, Linux x86_64 AppImage e Linux arm64 AppImage.

**Correção no V5:** novo **Gate N — Packaged artifact localization**. Ele verifica recursos/localizações no artifact realmente produzido e detecta sobrescrita posterior por packaging.

## Hardening incorporado ao plano

O `PLANO_MESTRE_V5.md` integra os achados acima diretamente em:

- ownership/classes de inventário;
- arquitetura Flutter e services;
- Android native/PiP/accessibility;
- Linux/Web/Windows;
- reachability e provenance;
- fases 0/1/5/7/11/13;
- Gates D/G/H e novo Gate N;
- matriz de testes;
- riscos/mitigações;
- Definition of Done;
- evidência mínima por PR.

## Critério de completude V5

Um shipping locale não é considerado completo apenas porque ARB e resources fonte estão em 100%. Para PT-BR, a evidência final deve cobrir:

    ARB
    native resources
    framework-owned localized widgets
    app-supplied system UI
    remote product copy
    product-controlled remote catalog
    runtime asset copy
    build-generated product copy
    packaged artifact localization
    official product content
    runtime visual assets
    approved external/user data

Assim, o conceito de 100% passa a significar: **toda copy própria alcançável possui owner, locale policy, mecanismo de resolução e evidência no runtime/artifact em que realmente aparece**.

## Resultado

`PLANO_MESTRE_V5.md` substitui a V4 como especificação canônica.

V1, V2, V3 e V4 permanecem preservadas como histórico/audit trail.