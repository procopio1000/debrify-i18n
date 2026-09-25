# Plano Mestre V1 — Internacionalização (i18n/l10n) completa do Debrify

**Projeto de planejamento:** `procopio1000/debrify-i18n`  
**Projeto-alvo upstream:** `varunsalian/debrify`  
**Baseline auditada:** `v0.10.0-beta.1`  
**Commit-alvo da auditoria:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`  
**Data da auditoria:** 2026-09-25  
**Idioma inicial prioritário:** Português do Brasil (`pt-BR`)  
**Objetivo estrutural:** criar uma base de internacionalização robusta e extensível para qualquer idioma, sem acoplar idioma da interface a idioma de metadados, áudio ou legendas.

---

## 0. Resumo executivo

O Debrify v0.10.0-beta.1 ainda não possui uma infraestrutura de localização da interface baseada no sistema oficial do Flutter. A auditoria encontrou `intl` já presente como dependência, porém não encontrou `flutter_localizations`, `supportedLocales`, `localizationsDelegates`, `AppLocalizations`, arquivos ARB ou uma pasta `l10n` dedicada à UI.

A internacionalização deve ser tratada como uma mudança transversal de arquitetura, não como uma substituição pontual de textos. O projeto possui UI Flutter ampla, múltiplos fluxos de bootstrap, UI nativa Android TV, telas de TV com D-pad, player, perfis, onboarding, settings, IPTV, Debrify TV, Stremio TV, integrações de debrid, rastreadores, WebDAV, backup, sincronização, acesso remoto e várias superfícies de mensagens de erro/status.

A estratégia recomendada é:

1. Introduzir `flutter_localizations` + `gen_l10n` + ARB como fonte única de textos estáticos da UI.
2. Adicionar inglês como template canônico e `pt-BR` como primeira tradução completa.
3. Criar um controlador de idioma da interface separado de metadados/áudio/legendas.
4. Migrar por domínios funcionais, com CI impedindo novas strings de UI hardcoded.
5. Tratar Android nativo, tvOS/iOS/macOS, desktop e Web como superfícies próprias.
6. Adotar ICU para pluralização, placeholders, números, datas e mensagens condicionais.
7. Preparar layout, acessibilidade, fontes e navegação para idiomas longos e RTL desde o primeiro desenho.
8. Garantir 100% de cobertura para qualquer idioma declarado como suportado em release.
9. Fazer a mudança sem alterar protocolos, IDs persistidos, nomes de provedores, nomes de addons, títulos de mídia, nomes de arquivos/torrents ou dados vindos de APIs.

---

# 1. Baseline da auditoria

## 1.1 Estrutura do repositório auditado

A árvore do commit auditado possui aproximadamente:

- 3.666 entradas na árvore Git.
- 1.757 arquivos Dart.
- 508 arquivos Dart diretamente sob `lib/screens`, `lib/widgets` e `lib/theme`.
- 614 testes Dart.
- 743 arquivos sob diretórios de plataforma (`android`, `ios`, `macos`, `linux`, `windows`).
- Um único `MaterialApp` principal em `lib/main.dart`, mas existem vários `MaterialApp` auxiliares de bootstrap/recovery dentro do mesmo arquivo.

Os hubs mais importantes, conforme o próprio `CODEMAP.md`, incluem:

- `lib/main.dart`
- `lib/screens/search_screen.dart`
- `lib/services/storage_service.dart`
- `lib/screens/video_player_screen.dart`
- `lib/screens/magic_tv_screen.dart`
- `lib/screens/torbox/torbox_downloads_screen.dart`
- `lib/screens/debrid_downloads_screen.dart`
- `lib/widgets/initial_setup_flow.dart`
- `lib/services/video_player_launcher.dart`
- `lib/services/torrent_playback_service.dart`

A implementação deve respeitar o aviso do próprio projeto de não ler/alterar arquivos gigantes sem busca focada.

## 1.2 Evidências de strings de interface espalhadas

A busca estática no commit auditado encontrou, em nível de arquivos contendo os padrões:

- cerca de 170 arquivos com `Text('...')`;
- cerca de 281 arquivos com campos `title:`;
- cerca de 326 arquivos com campos `label:`;
- cerca de 147 arquivos com campos `subtitle:`;
- cerca de 56 arquivos com `tooltip:`;
- cerca de 55 arquivos com `hintText:`;
- cerca de 115 arquivos contendo `SnackBar(`;
- cerca de 77 arquivos contendo `AlertDialog(`;
- cerca de 32 arquivos contendo `Semantics(`;
- cerca de 15 arquivos com `errorText:`;
- cerca de 152 arquivos contendo `this.label`, indicando diversos modelos/enums com rótulos embutidos;
- cerca de 89 arquivos com `displayName`;
- cerca de 105 arquivos com `description`.

Esses números são indicadores de superfície e **não equivalem ao número exato de mensagens**, pois cada arquivo pode conter várias ou nenhuma string de UI relevante para tradução.

## 1.3 Estado atual da localização Flutter

No baseline não foram encontrados:

- `flutter_localizations`;
- `AppLocalizations`;
- `supportedLocales`;
- `localizationsDelegates`;
- configuração `l10n.yaml`;
- arquivos `app_*.arb`;
- infraestrutura de tradução da UI.

O `pubspec.yaml` já contém `intl`, mas isso hoje não representa localização da interface.

## 1.4 Linguagem de UI não deve ser confundida com linguagem de conteúdo

O projeto já possui conceitos de idioma em áreas distintas:

- `MetadataPreferences.language`
- `artworkLanguage`
- `trailerLanguage`
- preferências de áudio
- preferências de legenda
- `LanguageMapper.kt` no player Android nativo
- preferência específica por Português Brasileiro para legendas

Esses conceitos devem permanecer independentes da linguagem da UI.

**Invariante:** mudar `App language` nunca deve, por si só:

- trocar idioma de áudio;
- trocar idioma de legenda;
- trocar idioma de metadados;
- trocar região de disponibilidade;
- alterar filtros;
- alterar ordenação de fontes;
- modificar comportamento de addons.

## 1.5 Bootstrap possui múltiplas superfícies críticas

`lib/main.dart` possui múltiplos fluxos que podem renderizar antes da aplicação principal:

- recovery do registry/perfis;
- `_StartupFailureApp`;
- `_MigrationUpdateScreen`;
- `_LinuxVaultBootstrapHost`;
- `DebrifyApp` principal.

A internacionalização só estará completa se **todos** esses caminhos tiverem delegates/supportedLocales e puderem renderizar no idioma correto, inclusive em falha de inicialização.

## 1.6 UI nativa Android TV

`android/app/src/main/res/values/strings.xml` já contém algumas strings do player, como:

- Audio Tracks
- Subtitle Tracks
- Rewind 10 seconds
- Play or pause
- Next
- Audio
- Subtitles
- Aspect
- Fetching next stream

Porém há também strings de UI hardcoded em Kotlin, especialmente no navegador de fontes nativo, por exemplo conceitos como:

- All sources
- Other sources
- Pinned source
- source/sources
- Looking for season packs...
- DIRECT / EXTERNAL / TORRENT
- seeders

Portanto o plano precisa incluir recursos Android nativos, não apenas ARB Flutter.

## 1.7 Formatação atual

`lib/utils/formatters.dart` possui datas em padrão fixo como:

- `MMM dd, yyyy`
- `MMM dd, yyyy HH:mm`

e tamanhos com decimal gerado por `toStringAsFixed`, o que não respeita automaticamente convenções locais de separador decimal.

Isso deve ser migrado para formatação orientada por locale.

---

# 2. Princípios arquiteturais obrigatórios

## 2.1 Fonte única para texto de UI

Todo texto de interface estático deve vir de `AppLocalizations`, salvo exceções documentadas.

Não usar:

```dart
Text('Settings')
```

Usar:

```dart
Text(context.l10n.settingsTitle)
```

## 2.2 Identificadores nunca são traduzidos

Não traduzir:

- IDs de banco;
- chaves SharedPreferences;
- nomes de enum usados em persistência;
- valores de protocolo;
- endpoints;
- `sourceId`;
- `addonId`;
- nomes de providers;
- MIME types;
- codecs;
- nomes reais de serviços/brands;
- IDs de eventos de analytics/diagnóstico;
- nomes de campos JSON;
- nomes de rotas;
- tokens como `directUrl`, `torrent`, `fetched`, `failed`.

A UI apresenta uma tradução para o usuário, mas a lógica continua usando valores canônicos estáveis.

## 2.3 Dados externos não são traduzidos localmente

Por padrão não traduzir:

- títulos de filmes/séries recebidos do provider;
- sinopses recebidas do provider;
- nomes de addons;
- nomes de arquivos;
- nomes de torrents/releases;
- nomes de servidores;
- nomes de playlists;
- nomes definidos pelo usuário;
- mensagens arbitrárias vindas de servidores.

Para metadados, pedir o idioma correto ao provider através das preferências já existentes.

## 2.4 Localization não entra em modelos de domínio

Evitar `BuildContext` ou `AppLocalizations` dentro de models/services.

Enums como:

```dart
enum MetadataCategory {
  information('Titles and descriptions');
}
```

devem evoluir para representação canônica sem texto localizado:

```dart
enum MetadataCategory { information, posters, backgrounds, ... }
```

e a camada de apresentação faz:

```dart
String metadataCategoryLabel(AppLocalizations l10n, MetadataCategory value) {
  return switch (value) {
    MetadataCategory.information => l10n.metadataCategoryInformation,
    ...
  };
}
```

Caso remover `.label` imediatamente gere refactor excessivo, usar fase transitória:

- manter `.label` apenas como fallback/debug;
- proibir novas leituras da propriedade em widgets;
- migrar UI para mapper localizado;
- remover `.label` em fase posterior.

## 2.5 Preferência de idioma é local ao dispositivo

Recomendação inicial:

- `App language` é uma preferência local do dispositivo;
- não pertence ao perfil;
- não é sincronizada via WebDAV;
- não é importada automaticamente entre dispositivos;
- não é alterada ao trocar de perfil;
- deve funcionar antes da seleção/desbloqueio do perfil.

Motivo: a UI precisa de idioma antes do `ProfileGate`, e um mesmo perfil pode ser usado em dispositivos de pessoas/ambientes diferentes.

Uma futura opção per-profile pode ser adicionada depois, mas não deve complicar V1.

---

# 3. Arquitetura alvo

## 3.1 Dependências

Em `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter

  flutter_localizations:
    sdk: flutter

  intl: ^0.19.0
```

Manter a versão de `intl` compatível com a versão de Flutter efetivamente usada pelo projeto.

Em:

```yaml
flutter:
  generate: true
```

## 3.2 Estrutura de arquivos

```text
lib/
  l10n/
    app_en.arb
    app_pt_BR.arb
    app_localizations.dart        # gerado
    app_localizations_en.dart     # gerado
    app_localizations_pt.dart     # gerado, conforme gen_l10n
    l10n_extensions.dart
    l10n_mappers.dart
    locale_option.dart
    locale_resolution.dart
  services/
    app_locale_controller.dart
  screens/
    settings/
      app_language_page.dart

l10n.yaml

tool/
  l10n_audit.dart
  l10n_validate_arb.dart

test/
  l10n/
    app_locale_controller_test.dart
    locale_resolution_test.dart
    arb_completeness_test.dart
    hardcoded_ui_strings_test.dart
    bootstrap_localization_test.dart
    formatting_localization_test.dart
    rtl_layout_contract_test.dart
    pseudo_locale_layout_test.dart
```

## 3.3 `l10n.yaml`

Configuração recomendada:

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
synthetic-package: false
required-resource-attributes: true
nullable-getter: false
use-named-parameters: true
format: true
use-escaping: true
untranslated-messages-file: build/l10n/untranslated_messages.json
preferred-supported-locales:
  - en
  - pt_BR
```

Antes de implementar, validar flags contra o Flutter 3.44.8 efetivamente usado pelo workflow.

## 3.4 Extensão de acesso

```dart
extension AppLocalizationsX on BuildContext {
  AppLocalizations get l10n => AppLocalizations.of(this);
}
```

Objetivo:

- reduzir boilerplate;
- facilitar review;
- padronizar migração.

## 3.5 `AppLocaleController`

Responsabilidades:

- carregar override persistido;
- expor `Locale?`, onde `null` significa “System default”;
- salvar `pt-BR`, `en`, etc.;
- validar locale contra lista suportada;
- notificar a raiz imediatamente;
- tolerar preferência corrompida;
- nunca bloquear startup de forma fatal.

API sugerida:

```dart
class AppLocaleController extends ChangeNotifier {
  static final instance = AppLocaleController._();

  Locale? get overrideLocale;
  bool get followsSystem;

  Future<void> initialize();
  Future<void> useSystemLocale();
  Future<void> setLocale(Locale locale);
}
```

Persistência sugerida:

```text
ui_locale_v1 = system | en | pt-BR | ...
```

ou ausência da key = sistema.

## 3.6 Resolução de locale

Ordem:

1. override manual válido;
2. locale exato do sistema;
3. idioma + script compatível;
4. idioma compatível;
5. inglês.

Casos obrigatórios:

- `pt-BR` → `pt_BR`;
- `pt_BR` persistido legado → normalizar para `pt-BR`;
- `pt` sem região → usar política explícita;
- `pt-PT` não deve fingir que é `pt-BR` se no futuro existir tradução própria;
- locale desconhecido → inglês;
- preferência inválida → limpar/fallback sem crash.

Recomendação V1:
- `en`
- `pt_BR`

O seletor sempre mostra autônimos:
- System default
- English
- Português (Brasil)

Não usar bandeiras como representação principal de idioma.

## 3.7 MaterialApp principal

No `MaterialApp` principal:

```dart
locale: AppLocaleController.instance.overrideLocale,
localizationsDelegates: AppLocalizations.localizationsDelegates,
supportedLocales: AppLocalizations.supportedLocales,
localeListResolutionCallback: resolveDebrifyLocale,
```

A raiz deve observar o controller para rebuild imediato.

## 3.8 MaterialApp auxiliares

Repetir suporte de localização para:

- recovery;
- startup failure;
- migration update;
- Linux vault bootstrap;
- qualquer host de teste/preview que renderize UI real.

Criar helper de configuração compartilhada para evitar drift, sem tentar refatorar todo bootstrap em uma única árvore.

Exemplo conceitual:

```dart
class DebrifyLocalizationConfig {
  static final delegates = AppLocalizations.localizationsDelegates;
  static final supportedLocales = AppLocalizations.supportedLocales;
}
```

Evitar grandes refactors no fluxo de inicialização apenas para adicionar i18n.

---

# 4. ARB — padrão de qualidade

## 4.1 Inglês é template canônico

`app_en.arb` é a fonte de contrato.

Todos os idiomas de release precisam possuir 100% das keys do template, exceto mensagens explicitamente marcadas como não aplicáveis por política — preferencialmente nenhuma.

## 4.2 Nomenclatura semântica

Evitar:

```json
"save": "Save"
```

quando o contexto pode variar.

Preferir:

```json
"profileEditSaveButton": "Save",
"profileCreateTitle": "Create profile",
"settingsLanguageTitle": "App language"
```

Padrão:

```text
<domínio><tela/componente><elemento/ação/estado>
```

Exemplos:

- `profilesTitle`
- `profileEditRoleAdminTitle`
- `profileEditRoleAdminDescription`
- `profileEditPinHint`
- `settingsLanguageSystemDefault`
- `playerAudioTracksTitle`
- `sourcesCount`
- `downloadsDeleteConfirmTitle`
- `syncDeviceLastSeen`
- `errorNetworkUnavailable`

## 4.3 Metadata obrigatório por key

Cada mensagem deve ter descrição.

```json
{
  "profileEditSaveButton": "Save",
  "@profileEditSaveButton": {
    "description": "Button in Edit profile that saves profile changes."
  }
}
```

Para placeholders:

```json
{
  "sourcesCount": "{count, plural, =0{No sources} =1{1 source} other{{count} sources}}",
  "@sourcesCount": {
    "description": "Number of available playback sources.",
    "placeholders": {
      "count": {
        "type": "int",
        "example": "12"
      }
    }
  }
}
```

## 4.4 Nunca concatenar sentenças traduzidas

Não:

```dart
'Delete ' + count.toString() + ' downloads?'
```

Usar uma única mensagem ICU.

Também não construir frases juntando rótulos de partes que podem mudar de ordem em outros idiomas.

## 4.5 Pluralização

Usar ICU para:

- source/sources;
- addon/addons;
- episode/episodes;
- season/seasons;
- torrent/torrents;
- download/downloads;
- device/devices;
- result/results;
- channel/channels;
- item/items;
- minute/minutes;
- second/seconds;
- file/files;
- seed/seeders, quando exibido como frase;
- watched counts;
- search counts.

## 4.6 Placeholders

Placeholders para:

- nomes de perfis;
- serviços;
- provedores;
- títulos;
- dispositivos;
- contagens;
- tempo;
- tamanho;
- versão;
- erro categorizado;
- porcentagens;
- atalhos de teclado.

Nunca inserir mensagens de exceção brutas sem política de redaction.

## 4.7 Mensagens de erro

Separar:

1. **reason code canônico** interno;
2. mensagem localizada segura para UI;
3. detalhe técnico redigido para diagnostics/log.

Exemplo:

```dart
enum UiErrorReason {
  networkUnavailable,
  authenticationExpired,
  resourceUnavailable,
  unknown,
}
```

Mapear para ARB.

Não tentar traduzir `Exception.toString()`.

---

# 5. Formatação internacional

## 5.1 Datas

Remover padrão fixo como `MMM dd, yyyy` para UI geral.

Preferir:

```dart
DateFormat.yMMMd(localeName).format(date)
```

e variantes local-aware.

## 5.2 Horas

Usar formato dependente de locale/sistema quando apropriado.

Não forçar 12h/24h em texto de UI se o SO tem preferência aplicável.

## 5.3 Números

Usar `NumberFormat` para:

- contagens;
- percentuais;
- velocidades;
- ratings, quando exibidos numericamente;
- estatísticas.

## 5.4 Tamanho de arquivos

Preservar unidades técnicas (`KB`, `MB`, `GB`) quando desejado, porém formatar o número conforme locale.

Exemplo pt-BR:

```text
1,5 GB
```

e não:

```text
1.5 GB
```

## 5.5 Duração

Não montar:

```text
10 seconds
```

manualmente.

Usar mensagens ICU de duração para UI textual.

Para timestamps técnicos/logs, manter formato técnico.

## 5.6 Relative time

Auditar todo código que monta:

- `x min ago`
- `today`
- `yesterday`
- `last seen`
- `updated ... ago`

e mover para l10n/Intl.

---

# 6. Matriz de migração por domínio

Cada fase deve fechar com testes antes da próxima.

## Fase 0 — Fundação e guardrails

Arquivos/áreas:

- `pubspec.yaml`
- `l10n.yaml`
- `lib/l10n/*`
- `lib/services/app_locale_controller.dart`
- `lib/main.dart`
- `analysis_options.yaml`
- `tool/l10n_audit.dart`
- testes `test/l10n/*`

Entregas:

- `gen_l10n` funcionando;
- inglês + pt-BR;
- seletor básico;
- persistência local;
- todos os `MaterialApp` com delegates;
- CI mínimo;
- scanner de novas strings.

Aceite:

- mudar para pt-BR sem reiniciar;
- fechar/abrir app preserva override;
- `System default` reage ao sistema;
- startup failure e recovery não quebram.

## Fase 1 — Navegação, perfis, onboarding e Settings

Prioridade máxima porque são as telas de configuração para não falantes de inglês.

Cobrir:

- `lib/main.dart` navegação;
- `lib/widgets/mobile_*_nav.dart`;
- `lib/widgets/desktop_*_nav.dart`;
- `lib/widgets/tv_sidebar_nav.dart`;
- `lib/screens/profiles/*`;
- `lib/widgets/profiles/*`;
- `lib/widgets/onboarding/*`;
- `lib/widgets/initial_setup_flow.dart`;
- `lib/screens/settings_screen.dart`;
- `lib/screens/settings/*`;
- `lib/screens/settings/widgets/*`;
- busca de Settings.

Pontos especiais:

- Role: Admin / Member / Kid;
- descrições de permissões;
- PIN/lock/autolock;
- Access;
- diagnostics;
- sync/migrate;
- textos de conectividade;
- mensagens de erro de configuração;
- todos os tooltips;
- semântica/a11y;
- hints de teclado TV.

## Fase 2 — Home, Discover, Search, Details e Collections

Cobrir:

- `search_screen.dart`;
- `browse_screen.dart`;
- `metadata_explore_page.dart`;
- `catalog_item_detail_screen.dart`;
- `merged_series_detail_screen.dart`;
- `widgets/home/*`;
- `widgets/detail/*`;
- `widgets/search_*`;
- `screens/search/*`;
- `screens/see_all/*`;
- `screens/collections/*`;
- `widgets/collections/*`.

Não traduzir dados de catálogo localmente.

Localizar:

- headings estruturais;
- Empty states;
- “See all”;
- filtros;
- menus;
- ações de watch progress;
- Continue Watching;
- mensagens de carregamento/erro;
- labels de pessoas/estúdios/disponibilidade.

## Fase 3 — Addons, engines, fontes e filtros

Cobrir:

- `screens/addons/*`;
- `addons_screen.dart`;
- `services/engine/*` somente strings de UI expostas;
- `widgets/source_row.dart`;
- `widgets/cinema_sources_layout.dart`;
- `widgets/add_source_picker_dialog.dart`;
- `screens/video_player/widgets/source_sheet.dart`;
- filter settings;
- torrent engine settings;
- indexer managers;
- stream badges settings.

Atenção:

- nomes de addons = dados, não tradução;
- nomes de trackers/indexers = dados;
- qualidades `2160p`, `1080p` = tokens técnicos;
- codecs = tokens técnicos;
- `WEB-DL`, `REMUX`, `HDR`, `DV` = tokens técnicos;
- descrições do app em torno desses itens = localizadas.

## Fase 4 — Debrid, cloud e downloads

Cobrir:

- Real-Debrid;
- TorBox;
- Premiumize;
- AllDebrid;
- PikPak;
- WebDAV;
- `cloud_screen.dart`;
- downloads screens;
- account status widgets;
- bind/add/download dialogs;
- folder pickers;
- erros de autenticação;
- confirmações destrutivas.

Proibir tradução de:

- nomes dos provedores;
- nomes de arquivos;
- nomes de pastas do usuário;
- mensagens de servidor sem normalização.

## Fase 5 — Player Flutter

Cobrir:

- `video_player_screen.dart`;
- `screens/video_player/widgets/*`;
- controls;
- source sheet;
- channel guide;
- dock;
- audio/subtitle menu;
- sync;
- skip segment;
- playback errors;
- shuffle/random;
- sleep timer;
- next episode;
- reconnect/recovery notices.

Requisitos TV:

- não quebrar D-pad;
- texto longo não deslocar foco;
- overlays continuam dentro da safe area;
- labels de controles possuem Semantics localizados.

## Fase 6 — Android TV player nativo

Mover toda UI Kotlin para recursos Android:

```text
android/app/src/main/res/values/strings.xml
android/app/src/main/res/values-pt-rBR/strings.xml
```

Auditar:

- `AndroidTvTorrentPlayerActivity.kt`;
- `TvSourceBrowserController.kt`;
- controles;
- source picker;
- badges built-in;
- erros;
- hints;
- contentDescription;
- live/IPTV;
- next stream;
- track selectors.

### Sincronização de locale Flutter → player nativo

Problema crítico:
um override manual dentro do Flutter não garante automaticamente que um `Activity` nativo use o mesmo locale.

Implementar contrato explícito:

```text
uiLocale = "system" | BCP47
```

ao lançar o player nativo.

Opções técnicas:

1. passar `uiLocale` como extra no Intent;
2. criar Context/Resources com locale correspondente no player;
3. centralizar em `NativeLocaleBridge`.

Não depender somente do locale global do aparelho quando o usuário escolheu override dentro do Debrify.

Testar:

- System + Android pt-BR;
- override English com sistema pt-BR;
- override pt-BR com sistema English.

## Fase 7 — IPTV / Debrify TV / Stremio TV

Cobrir:

- `widgets/iptv/*`;
- `screens/settings/iptv_*`;
- `screens/debrify_tv/*`;
- `magic_tv_screen.dart`;
- `screens/stremio_tv/*`;
- guide sheets;
- channel pickers;
- import/export;
- catch-up/start-over;
- favorites;
- custom lists;
- EPG states;
- recording.

Não traduzir:

- nome de canal;
- nome de programa vindo do EPG;
- nome de categoria da playlist, salvo categorias próprias do app;
- URLs;
- playlist names do usuário.

## Fase 8 — Tracking, comunidade e fontes auxiliares

Cobrir:

- Trakt;
- Simkl;
- MDBList;
- YouTube;
- Reddit;
- Lemmy;
- calendar;
- watched/unwatched;
- scrobbling statuses;
- list management;
- save/remove actions.

Marcas permanecem intactas.

## Fase 9 — Sync, backup, remote e recovery

Cobrir:

- WebDAV sync;
- backup local;
- restore;
- migrate;
- remote pairing;
- remote keyboard;
- transfer progress;
- device management;
- recovery;
- vault Linux;
- startup/migration surfaces.

Essas telas são críticas: um erro de tradução não pode bloquear recuperação de dados.

## Fase 10 — Plataformas não-Flutter

### Android

- `values/strings.xml`
- `values-pt-rBR/strings.xml`