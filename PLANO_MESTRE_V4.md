# Plano Mestre V4 — Internacionalização (i18n/l10n) completa do Debrify

**Status:** versão consolidada, quarta auditoria extrema, implementation-ready, toolchain-verified e cross-runtime-verified  
**Projeto de planejamento:** procopio1000/debrify-i18n  
**Projeto-alvo upstream:** varunsalian/debrify  
**Baseline verificada:** v0.10.0-beta.1  
**Commit-alvo:** 9619c10b06ee919cacbe996b30be7739dc09c6d6  
**Data da reauditoria:** 2026-09-25  
**Primeiro locale de release:** Português do Brasil (pt-BR)  
**Template canônico:** inglês (en)  
**Escopo:** Flutter + Android nativo/TV + Services/Receivers + iOS + tvOS + macOS + Windows + Linux + Web + packages runtime + acessibilidade + formatação + busca + CI + release  
**Escopo estrutural medido na V3:** 1.574 arquivos de runtime nos roots principais do artefato auditado (contagem baseline; deve ser recalculada por commit)

---

# 0. Resultado das reauditorias V1 → V2 → V3 → V4

O V1 estava na direção correta, mas não estava completo o suficiente para ser tratado como plano final.

## 0.1 Falha estrutural encontrada no próprio V1

O arquivo PLANO_MESTRE_V1.md termina no começo da Fase 10:

- Fase 10 — Plataformas não-Flutter
- Android
- values/strings.xml
- values-pt-rBR/strings.xml

Não há conclusão da Fase 10, não há fases finais de hardening/release, não há estratégia de rollout/upstream, não há Definition of Done consolidada e não há fechamento de riscos.

O V2 corrigiu o truncamento estrutural do V1. A V3 fechou os blockers de toolchain, native background, ownership de locale e reachability. A V4 preserva V1–V3 como histórico e passa a ser a especificação canônica, adicionando auditoria de copy remota controlada pelo produto, assets de runtime, notificações de plugin fora do Android, rich text/custom painting e hardcoded directionality.

## 0.2 Baseline upstream reconfirmada

Em 2026-09-25 a release mais recente do Debrify continua sendo v0.10.0-beta.1 e o HEAD verificado continua no commit 9619c10b06ee919cacbe996b30be7739dc09c6d6.

Também foi reconfirmado no código:

- Flutter 3.44.8 no workflow;
- intl ^0.19.0;
- shared_preferences ^2.2.2;
- ausência de flutter_localizations;
- ausência de AppLocalizations;
- ausência de supportedLocales;
- ausência de infraestrutura ARB de UI;
- formatação fixa de data em lib/utils/formatters.dart;
- recursos Android parciais em values/strings.xml;
- várias strings user-facing em Dart, Kotlin/Java e superfícies de plataforma.

## 0.3 Lacunas importantes que o V1 não tornava explícitas

A reauditoria encontrou pontos que passam a ser requisitos de primeira classe:

1. SettingsRows guarda títulos/subtítulos em static const String. Isso impede simplesmente substituir Text('...') por context.l10n. O registry de Settings precisa ser refatorado para separar identidade estável de copy localizada.
2. Settings Search indexa title/subtitle/category/keywords e acrescenta literalmente a palavra settings ao haystack. A busca precisa ser reconstruída no locale ativo e possuir normalização apropriada.
3. Existem comparações de strings visíveis que participam da lógica. Exemplo confirmado: source_sheet.dart usa All sources como rótulo e também o compara para decidir apresentação. Display text nunca pode funcionar como identificador de controle.
4. Há textos user-facing gerados fora de widgets, inclusive em utils/services. O scanner não pode procurar apenas Text(...), SnackBar e AlertDialog.
5. Há transformações toUpperCase em dados visíveis. É necessário distinguir uppercase técnico de transformação de copy/dados externos.
6. Android possui mais de uma Activity nativa de player. A política de locale precisa cobrir todas, não apenas um player.
7. iOS/macOS possuem NSLocalNetworkUsageDescription em inglês. Essas mensagens pertencem ao SO e precisam de localização nativa.
8. tvOS possui Runner e Top Shelf extension. A extensão também entra no inventário.
9. Web ainda contém descrição placeholder A new Flutter project no manifest e index.html. O shell web precisa ser auditado.
10. O teclado Debrify para TV possui página de letras e uppercase próprios. A arquitetura futura não pode presumir que todo idioma usa somente o alfabeto inglês.
11. Shipping locales precisam ser separados dos locales gerados durante desenvolvimento, para impedir que uma tradução parcial vire idioma oficialmente selecionável.
12. O V1 não fechava a estratégia de rollout, PRs upstream, rollback, observabilidade e critérios mensuráveis de conclusão.

## 0.4 Novos achados bloqueadores da auditoria V3

A V3 adiciona requisitos que não estavam suficientemente fechados no V2:

1. **Conflito de dependência real no primeiro PR.** O SDK Flutter 3.44.8 declara `flutter_localizations -> intl 0.20.2`, enquanto o Debrify declara `intl ^0.19.0`. A fundação i18n deve atualizar a dependência direta de `intl` no mesmo PR; não usar `dependency_overrides` para mascarar incompatibilidade.
2. **`synthetic-package` já foi removido.** No Flutter 3.44.8, `synthetic-package: false` não tem efeito e gera warning; `true` é rejeitado. A chave não deve existir em `l10n.yaml`.
3. **O bridge Android por Activity é insuficiente.** Downloads, gravações, alarmes e receivers podem produzir UI/notificações quando o processo Flutter está morto. É necessário um espelho nativo durável do locale, com o Dart permanecendo a autoridade do produto.
4. **Há lógica nativa baseada em frases inglesas.** `MediaStoreDownloadService` e `LiveRecordingService` inferem estados a partir de strings como `Download complete`, `Saved...`, `Starting...` e `Recording finished...`. Esses branches devem virar enums/reason codes antes de localizar a copy.
5. **O Android possui grande superfície XML hardcoded.** A baseline auditada contém centenas de XMLs de runtime e dezenas de layouts próprios de TV/player com `android:text` e `android:contentDescription` literais. Migrar somente `values/strings.xml` não é suficiente.
6. **Notification channels são parte do contrato.** Nome/descrição de canais, ações e summaries precisam de resources localizados e re-upsert seguro quando o locale efetivo muda; o teste deve cobrir canal já existente.
7. **Apple tem superfícies controladas pelo SO.** `InfoPlist.strings`, permission prompts e seleção de bundle localization seguem o locale do sistema/per-app language, não um override interno arbitrário do Flutter. O plano precisa documentar essa fronteira em vez de prometer sincronização impossível.
8. **macOS ainda tem menu nativo inglês.** `macos/Runner/Base.lproj/MainMenu.xib` contém menus completos (About, Preferences, Edit, Find, Window, Help etc.). PT-BR exige localização do menu sem alterar selectors/shortcuts.
9. **Windows tem duas pendências concretas.** `windows/installer.iss` só declara inglês e possui copy de instalador hardcoded; `windows/runner/Runner.rc` ainda contém metadata template `com.example`.
10. **Web precisa de locale no DOM, não só copy Flutter.** `document.documentElement.lang` e `dir` devem acompanhar o locale/direção efetivos em runtime; `index.html` e manifest também não podem manter metadata placeholder.
11. **SourceSheet é um hotspot i18n.** Além de `All sources` como identidade visual, há plural manual, `toUpperCase()` em copy, `Playing` em Semantics e file-size decimal manual. A migração deve tratar o componente como unidade.
12. **Voice/input locale é uma autoridade separada.** O recognizer Android já aceita BCP-47. Idioma de ditado não deve ser acoplado automaticamente ao App language; deve ter política explícita.
13. **Scan repo-wide precisa conhecer reachability.** Código em `packages/` pode ser empacotado, enquanto `dev/`, exemplos, testes e generated code têm natureza diferente. Exclusão por diretório puro cria falso negativo; a classificação deve considerar se o código chega ao artefato.
14. **Locale matching regional precisa ser determinístico.** `pt-PT` não deve ser silenciosamente tratado como `pt-BR` só porque ambos começam com `pt`. O resolver deve ter tabela/política testada.
15. **Bidi isolation entra no contrato futuro.** Dados externos como URL, filename, release name e username interpolados dentro de copy RTL precisam de isolamento direcional para não corromper leitura ou ordem visual.

## 0.5 Novos achados bloqueadores da auditoria V4

A quarta auditoria percorreu não só source files tradicionais, mas também loaders de assets, configuração remota e sinks de renderização de texto que escapam de scanners focados em `Text(...)`.

1. **Existe copy user-facing controlada pelo produto fora do ARB.** `assets/config/app_remote_config.json` e `lib/services/support_remote_config_service.dart` carregam `settings_label`, `settings_subtitle`, título/mensagem de campanha e `button_label`, consumidos diretamente por `main.dart` e Settings. Sem contrato locale-aware, PT-BR pode ficar parcialmente inglês mesmo com ARB em 100%.
2. **O catálogo remoto de engines é outro contrato de localização.** Descrições exibidas no onboarding podem vir de `metadata.yaml` mantido pelo ecossistema oficial. Copy controlada pelo produto fora deste repositório não pode ser classificada automaticamente como “external data” e esquecida.
3. **Downloads não-Android possuem notificações próprias em Dart/plugin.** `lib/services/download_service.dart` configura `TaskNotification('Downloading'...)`, `Download complete`, `Download failed` e `Download paused`. A V3 endurecia Android notifications, mas esse caminho também é user-facing.
4. **Rich text é uma superfície relevante.** Existem literais em `Text.rich`/`TextSpan` como `Resume ·`, `Season`, `Community addons from` e `Tap to search`. O gate precisa reconhecer árvores de `InlineSpan`, não apenas widgets `Text`.
5. **Custom painters desenham copy.** Launch idents usam `TextPainter/TextSpan` e contêm copy como `PLAY · ANYTHING`, `LAUNCH` e outros rótulos. Texto desenhado em Canvas continua sendo interface e precisa ser localizado ou classificado conscientemente como brand/art direction.
6. **Assets de runtime precisam entrar no grafo de reachability.** JSON/YAML/Markdown/CSV ou qualquer arquivo carregado por `rootBundle`, asset manifest, loader próprio ou configuração remota pode produzir UI. “Scan de source code” sozinho não fecha 100% da superfície.
7. **Directionality hardcoded precisa de gate próprio.** Há `TextDirection.ltr` app-owned até em medição de texto de conteúdo, além de usos intencionais em branding/player. Cada ocorrência deve ser classificada como direção semântica, física/intencional ou bug; não basta uma revisão manual genérica de RTL.
8. **Rich text localizado precisa permitir reordenação gramatical.** Concatenar fragmentos estilizados em ordem inglesa pode gerar frases erradas em outros idiomas. Mensagens com spans clicáveis/estilizados devem usar placeholders semânticos ou componentes que permitam ao locale definir a ordem.
9. **Nome dos idiomas no seletor precisa de política explícita.** O seletor deve usar autônimos estáveis — por exemplo `English` e `Português (Brasil)` — e não traduzir o nome do idioma de destino conforme o locale atual, salvo decisão de UX documentada.
10. **Fallback inglês não conta como completude para copy remota própria.** Se um payload oficial não tiver PT-BR, ele pode cair para inglês por segurança, mas o release gate deve registrar isso como lacuna de localização, não como sucesso.
11. **Copy remota e dados de terceiros exigem trust boundaries diferentes.** Título de filme, filename, EPG e descrição fornecida por addon do usuário continuam dados externos; campanha, onboarding oficial e copy de suporte mantida pelo produto são responsabilidade de localização.
12. **A evidência de release precisa abranger runtime copy.** ARB/resource parity é necessário, mas não suficiente; o relatório de completude precisa somar ARB + native resources + product-owned remote/runtime copy + allowlists.
13. **Conteúdo oficial externo precisa de ownership explícito.** O app abre `https://debrify.tv/guides/webdav-sync/` e empacota um QR para a mesma URL. Mesmo fora do binário, uma instrução oficial acionada pela UI não deve cair silenciosamente na categoria “third-party data”.
14. **Release notes oficiais são renderizadas dentro do app.** `release.body` é exibido por `MarkdownBody` em `main.dart` e `settings_screen.dart`. Isso exige classificar Markdown/renderers e definir política para conteúdo editorial oficial.
15. **Assets visuais podem conter texto sem aparecer em source scan.** SVG pode ser analisado estruturalmente; PNG/JPG/PDF usados no runtime precisam de inventário visual/manual quando puderem carregar copy.

---

# 1. Objetivo e definição de sucesso

O projeto não é uma tradução PT-BR pontual. O objetivo é criar uma infraestrutura permanente de internacionalização e localização que permita adicionar idiomas sem reescrever arquitetura, sem misturar idioma de UI com idioma de conteúdo e sem introduzir regressões em TV, player, perfis, sync ou protocolos.

O primeiro release deve entregar PT-BR completo e inglês canônico.

Um locale só pode ser considerado shipping quando:

- 100% das mensagens exigidas estiverem traduzidas;
- placeholders e plurais forem equivalentes ao template;
- todas as superfícies críticas tiverem sido testadas;
- não houver strings user-facing conhecidas escapando dos mecanismos de localização, salvo allowlist justificada;
- layouts e navegação funcionarem com expansão de texto;
- recursos nativos correspondentes estiverem presentes;
- builds suportados passarem;
- o seletor de idioma estiver habilitado para aquele locale.

---

# 2. Invariantes arquiteturais obrigatórios

## 2.1 Idioma da UI é independente de conteúdo

App language nunca deve alterar automaticamente:

- idioma de metadados;
- idioma de artwork;
- idioma de trailers;
- idioma de áudio;
- idioma de legendas;
- região;
- filtros;
- ordenação de fontes;
- comportamento de addons;
- preferências de playback.

Essas preferências podem ter valores iguais, mas não compartilham autoridade.

## 2.2 Texto de apresentação nunca é identidade

É proibido usar texto localizado para:

- persistência;
- enum storage;
- comparação de lógica;
- branch de controle;
- analytics event id;
- cache key;
- database key;
- route id;
- source id;
- provider id;
- addon id;
- protocolo;
- deep link;
- intent action;
- JSON key.

Exemplo proibido:

    if (label == 'All sources') { ... }

Forma correta:

    if (group.id == SourceGroupId.all) { ... }

e somente a apresentação obtém a copy correspondente ao locale.

## 2.3 Dados externos permanecem dados

Não traduzir localmente por padrão:

- nomes de filmes/séries;
- sinopses vindas do provider;
- nomes de canais/EPG;
- nomes de arquivos;
- nomes de torrents/releases;
- nomes de addons;
- nomes de trackers/indexers;
- nomes de servidores;
- nomes de playlists do usuário;
- nomes de perfis;
- textos arbitrários recebidos de API.

Quando um provider suporta locale, o idioma deve ser solicitado ao provider usando a preferência de conteúdo apropriada, não o App language por acoplamento implícito.

## 2.4 Models e services não recebem BuildContext

A camada de domínio expõe:

- enums canônicos;
- reason codes;
- dados;
- estados;
- identificadores.

A camada de apresentação converte isso para AppLocalizations.

Exceptions brutas e Exception.toString() não são copy de UI.

## 2.5 Preferência de App language é installation/device setting em V3

Política inicial:

- local à instalação/dispositivo, fora do namespace de perfil;
- disponível antes do ProfileGate;
- não pertence ao perfil;
- usar `DevicePreferences`, não `ProfilePreferences`;
- registrar `ui_locale_v1` em `DevicePreferences.allowedKeys`;
- não introduzir raw `SharedPreferences.getInstance()` em feature code só para i18n;
- `ProfilePreferencePortability.allowsKey('ui_locale_v1')` deve retornar false;
- não é sincronizada por WebDAV;
- não é transportada pelo backup/restore de perfil do Debrify;
- não muda ao trocar de perfil;
- pode ser resetada para System default;
- preferência inválida/corrompida nunca pode impedir startup.

### Backup/migração do SO

“Device-local” aqui significa fora da autoridade de perfil/sync do Debrify.

Na baseline Android, `backup_rules.xml` e `data_extraction_rules.xml` excluem todos os SharedPreferences de cloud backup e device transfer; portanto `ui_locale_v1` não viaja pelo backup Android atual.

Em Apple/desktop, restore/migração de preferências pelo próprio SO é uma política da plataforma e não deve ser confundida com WebDAV/profile portability. Se o requisito futuro for “jamais atravessar instalação”, criar regra de plataforma explícita em vez de assumir comportamento.

## 2.6 Autoridade de locale por superfície

Não existe uma única API capaz de impor o override interno do Flutter a toda UI do sistema operacional.

Contrato V3:

| Superfície | Autoridade |
|---|---|
| Flutter UI | AppLocaleController |
| Android Activities próprias | AppLocaleController + NativeLocaleBridge; NativeLocaleStore lê a mesma preferência device-owned |
| Android Services/Receivers/notifications próprias | NativeLocaleStore, leitor nativo da mesma `ui_locale_v1` |
| Android system dialogs | locale efetivo do Android/app conforme plataforma |
| iOS/tvOS/macOS Flutter body | AppLocaleController |
| Apple permission prompts / InfoPlist.strings | bundle/per-app/system language do SO |
| macOS menu nativo | bundle localization; opcionalmente bridge explícito se upstream exigir seguir override interno |
| tvOS Top Shelf extension | bundle/system locale + dados/snapshot próprios da extensão |
| Windows installer | idioma do instalador/SO; ocorre antes de existir preferência do app |
| Web Flutter body | AppLocaleController |
| Web DOM `lang`/`dir` | espelho do locale efetivo do AppLocaleController |

A divergência inevitável de superfícies OS-owned deve ser documentada e testada; não criar hacks globais não suportados para “forçar” idioma do sistema.

## 2.7 Locale de input/voz é independente

Adicionar a matriz de autoridades:

    UI locale
    content locale
    audio/subtitle locale
    region
    input/speech locale

O TV voice recognizer aceita BCP-47. Política inicial recomendada:

- `input/speech locale = system` por padrão;
- App language não escreve essa preferência;
- se futuramente houver uma opção “Idioma da voz”, ela possui storage próprio;
- passar `EXTRA_LANGUAGE` somente quando houver escolha explícita ou regra de input documentada.

## 2.8 UI própria x UI de sistema/terceiros

Adicionar classificações de inventário:

- `SYSTEM_OWNED_UI`;
- `THIRD_PARTY_OWNED_UI`;
- `NATIVE_BACKGROUND_UI`;
- `GENERATED_CODE`.

Exemplos de `SYSTEM_OWNED_UI`: file picker, share sheet, botões do permission prompt. O app localiza somente a copy que realmente controla (por exemplo usage description).

Terceiros não entram em “100% de copy própria” sem prova de ownership, mas qualquer string que o Debrify injete neles continua em escopo.

## 2.9 Ownership de copy: local, remota oficial e externa

Toda string que chega à tela deve ter owner explícito. A V4 adiciona estas classificações de autoridade:

| Classe | Exemplos | Política |
|---|---|---|
| `LOCAL_PRODUCT_COPY` | botões, Settings, player chrome | ARB/native resources |
| `REMOTE_PRODUCT_COPY` | campanhas/suporte oficiais | payload locale-aware + fallback determinístico + completeness gate |
| `PRODUCT_CONTROLLED_REMOTE_CATALOG` | descrição oficial de engine/onboarding | schema locale-aware ou mapper local versionado |
| `THIRD_PARTY_EXTERNAL_DATA` | addon/EPG/provider externo | não traduzir localmente por padrão |
| `USER_DATA` | profile, filename, playlist | nunca traduzir automaticamente |
| `BRAND/TECHNICAL_TOKEN` | Debrify, Trakt, HDR, URL | preservar conforme glossário |
| `OFFICIAL_PRODUCT_CONTENT` | release notes, guia oficial, help/docs acionados pelo app | locale policy explícita; não confundir com third-party |
| `RUNTIME_VISUAL_ASSET` | PNG/JPG/SVG/PDF com texto embutido | localizar variante ou classificar como brand/technical |

Regras:

- copy oficialmente controlada pelo produto continua sendo responsabilidade de i18n mesmo quando vem da rede;
- fallback inglês é permitido para robustez, mas gera finding de completude em shipping locale;
- payload remoto nunca escolhe a autoridade de App language;
- locale solicitado/selecionado é enviado como BCP-47 somente quando o contrato remoto suporta locale;
- cache de remote config não pode “congelar” copy do locale anterior; preferir payload multilíngue versionado ou cache por locale;
- resposta remota desconhecida/malformada nunca impede startup;
- conteúdo oficial externo acionado pelo app declara se faz parte de CORE_UI_COMPLETENESS ou PRODUCT_EXPERIENCE_COMPLETENESS;
- setup/help essencial para concluir uma tarefa não pode ser excluído de cobertura apenas porque abre no browser.

Para `SupportRemoteConfig`, política preferida:

- mover `settings_label` e `settings_subtitle` estáveis para ARB;
- manter providers/URLs como dados;
- modelar campanha dinâmica como mapa por locale, por exemplo `copy.en` e `copy.pt-BR`;
- resolver exact BCP-47 -> regra regional explícita -> en;
- separar campaign id/timing/action URL da copy localizada.

## 2.10 Seletor de idioma usa autônimos

O language picker deve mostrar nomes estáveis definidos no catálogo de locales, não derivados de uma string traduzida pelo locale corrente.

Exemplo:

    en     -> English
    pt-BR  -> Português (Brasil)

Isso evita que o usuário precise entender o idioma atualmente ativo para conseguir trocar de idioma.

## 2.11 Dois níveis de completude

Para evitar uma definição ambígua de “100%”, a V4 separa:

### CORE_UI_COMPLETENESS — bloqueia ShippingLocales

Inclui:

- Flutter/native app chrome;
- messages/errors/settings;
- remote product copy renderizada como UI;
- runtime asset copy;
- notifications;
- accessibility;
- setup instructions indispensáveis renderizadas no app.

Meta: 100%.

### PRODUCT_EXPERIENCE_COMPLETENESS — cobertura oficial adjacente

Inclui:

- release notes oficiais exibidas em Markdown;
- guias oficiais abertos pelo app;
- website/help acionado diretamente por uma tarefa do app;
- store/release metadata quando controlada pelo projeto.

Para o fluxo WebDAV, como o app oferece `Setup guide` e um QR fixo para `debrify.tv/guides/webdav-sync/`, o destino deve ter política de locale. Preferir URL estável com content negotiation/locale picker ou URL derivada do locale efetivo. O QR não deve precisar ser regenerado por idioma se o landing page negociar locale.

Release notes podem ser publicadas em formato multilíngue ou via source locale-aware. Se o produto decidir que release notes são editoriais e não bloqueiam o shipping locale, essa exceção precisa ser explícita e mensurada — nunca implícita.


---

# 3. Arquitetura Flutter alvo

## 3.1 Stack

Adicionar:

    flutter_localizations:
      sdk: flutter

O Flutter 3.44.8 usado pelo upstream fixa `intl 0.20.2` em `flutter_localizations`.

No PR de fundação, alterar a dependência direta atual:

    intl: ^0.19.0

para uma constraint compatível com o pin do SDK, recomendada:

    intl: ^0.20.2

Sob Flutter 3.44.8, o lock deve resolver `intl 0.20.2`.

Regras:

- não usar `dependency_overrides` para forçar `intl`;
- rodar `flutter pub get` antes de qualquer migração de copy;
- revisar todos os usos existentes de `DateFormat` após a mudança;
- manter o lockfile coerente com o SDK pinado.

No bloco flutter:

    generate: true

Usar:

- gen_l10n;
- ARB;
- ICU MessageFormat;
- AppLocalizations gerado em source;
- extension BuildContext.l10n;
- AppLocaleController;
- mappers localizados na camada de apresentação;
- formatters locale-aware.

## 3.2 Estrutura recomendada

    lib/
      l10n/
        app_en.arb
        app_pt_BR.arb
        generated/
        l10n_extensions.dart
        l10n_mappers.dart
        locale_catalog.dart
        locale_codec.dart
        locale_resolution.dart
        localized_formatters.dart
        localized_search.dart
      services/
        app_locale_controller.dart
      screens/
        settings/
          app_language_page.dart

    l10n.yaml

    tool/
      l10n_audit.dart
      l10n_validate_arb.dart
      l10n_platform_audit.dart
      l10n_semantic_coupling_audit.dart

    test/
      l10n/
        ...

## 3.3 l10n.yaml

Configuração de referência:

    arb-dir: lib/l10n
    template-arb-file: app_en.arb
    output-dir: lib/l10n/generated
    output-localization-file: app_localizations.dart
    output-class: AppLocalizations
    required-resource-attributes: true
    nullable-getter: false
    use-named-parameters: true
    format: true
    use-escaping: true
    untranslated-messages-file: build/l10n/untranslated_messages.json
    preferred-supported-locales:
      - en
      - pt_BR

As flags acima foram rechecadas contra o toolchain Flutter 3.44.8.

Regras V3:

- **não** adicionar `synthetic-package`; a opção foi removida e `false` só gera warning;
- `required-resource-attributes`, `nullable-getter`, `format`, `use-escaping` e `use-named-parameters` existem no toolchain auditado;
- CI deve tratar warning inesperado de `gen-l10n` como regressão;
- cada ARB deve declarar `@@locale` coerente com o filename;
- locale de arquivo gerado usa convenção do gen_l10n (`pt_BR`); persistência do produto continua BCP-47 (`pt-BR`).

## 3.4 Shipping locales separados dos generated locales

Não usar AppLocalizations.supportedLocales diretamente como definição de produto durante migração.

Criar catálogo explícito:

    abstract final class ShippingLocales {
      static const locales = <Locale>[
        Locale('en'),
      ];
    }

Enquanto PT-BR estiver em desenvolvimento, o ARB pode existir para tradução/teste, mas o app público continua aceitando somente en.

Quando todos os gates de PT-BR passarem:

    Locale('pt', 'BR')

é promovido para ShippingLocales.

CI deve garantir:

- todo shipping locale existe nos ARBs gerados;
- nenhum locale não aprovado aparece no seletor;
- selector, locale resolution e MaterialApp usam ShippingLocales;
- debug/test pode habilitar pseudo/locales de desenvolvimento sem torná-los shipping.

Isso evita o problema de uma tradução parcial entrar automaticamente em AppLocalizations.supportedLocales.

## 3.5 AppLocaleController

Responsabilidades:

- initialize antes da UI que depende do locale;
- obter `DevicePreferences.instance()`;
- ler `ui_locale_v1` por `DevicePreferences`;
- null/system = seguir sistema;
- aceitar BCP 47;
- normalizar aliases legados;
- rejeitar locale não shipping;
- persistir atomicamente via `DevicePreferences`;
- manter `ui_locale_v1` em `DevicePreferences.allowedKeys`;
- não criar raw SharedPreferences access novo para o controller;
- notificar a raiz;
- reagir a mudança de locales do SO quando em System default;
- não bloquear startup em falha de storage.

API conceitual:

    class AppLocaleController extends ChangeNotifier
        with WidgetsBindingObserver {
      Locale? get overrideLocale;
      bool get followsSystem;

      Future<void> initialize();
      Future<void> useSystemLocale();
      Future<void> setLocale(Locale locale);
      void didChangeLocales(List<Locale>? locales);
    }

Persistência canônica única:

    DevicePreferences.allowedKeys += ui_locale_v1
    ui_locale_v1 = system | en | pt-BR | ...

Testes arquiteturais obrigatórios:

    ProfilePreferencePortability.allowsKey('ui_locale_v1') == false

Também ajustar os source-guards que pinam accesses device-owned para impedir regressão para storage profile-scoped ou raw SharedPreferences.

## 3.6 Codec e resolução de locale

Persistir BCP 47; construir Locale com languageCode/scriptCode/countryCode.

Ordem:

1. override manual shipping válido;
2. lista de locales do sistema na ordem do SO;
3. match exato language+script+region;
4. language+script;
5. language+region quando aplicável;
6. language;
7. fallback en.

Política V3 inicial:

- `en` é fallback e aceita variantes regionais `en-*`;
- `pt-BR` é explícito e shipping somente após promoção;
- `pt-PT` **não** faz language-only fallback para `pt-BR`;
- `pt` sem região pode mapear para `pt-BR` somente por regra explícita de produto, documentada e testada; na V3 a regra recomendada é `pt -> pt-BR` enquanto houver um único locale português shipping;
- valor legado `pt_BR` é normalizado para `pt-BR`;
- tags BCP-47 são canonicalizadas antes do match;
- valor desconhecido não causa crash e volta a system/en;
- o resolver customizado deve impedir que o fallback genérico do Flutter reintroduza `pt-PT -> pt-BR` acidentalmente.

Casos obrigatórios de teste: `pt-BR`, `pt_BR`, `pt`, `pt-PT`, `en-US`, `en-GB`, locale desconhecido e lista de preferências com múltiplos itens.

## 3.7 Todos os roots Flutter devem compartilhar a mesma configuração

A auditoria V3 confirmou múltiplos `MaterialApp` de produção em `lib/main.dart`.

Cobrir explicitamente:

- `DebrifyApp` principal;
- `_MigrationUpdateScreen`;
- `ProfileRecoveryScreen` hospedado pelo MaterialApp inline do recovery;
- `_StartupFailureApp`;
- `_LinuxVaultBootstrapHost`;
- qualquer novo host de bootstrap/recovery que possa executar antes do app principal.

### Ordem de startup

`AppLocaleController.initialize()` precisa ocorrer **antes de qualquer caminho que possa chamar runApp**.

Fluxo conceitual:

    WidgetsFlutterBinding.ensureInitialized()
      -> AppLocaleController.initialize()   // non-fatal, DevicePreferences
      -> migration/recovery/profile bootstrap decisions
      -> qualquer runApp

Se storage de locale falhar, o controller cai para system/en e o startup continua.

Criar um helper/wrapper compartilhado para os MaterialApps de bootstrap contendo:

- `locale`;
- `supportedLocales = ShippingLocales.locales`;
- delegates;
- locale resolution;
- listener do AppLocaleController.

O app principal mantém suas chaves/observers/builders próprios, mas consome a mesma fonte de locale.

Hosts de testes e ferramentas só precisam de localization quando renderizam widgets reais cuja copy depende dela.

Teste obrigatório: system en + override pt-BR mostra PT-BR até em migration/recovery/startup failure/Linux vault; system pt-BR + override en mostra inglês nessas mesmas superfícies.

## 3.8 Política de generated code

V3 escolhe uma política explícita para evitar drift:

- arquivos gerados por `gen_l10n` em `lib/l10n/generated/` **devem ser versionados**;
- o `.gitignore` atual do upstream não exclui esse diretório, portanto nenhuma exceção adicional é necessária na baseline;
- nunca são editados manualmente;
- todo PR que altera ARB/configuração executa `flutter gen-l10n` e inclui o diff gerado;
- CI executa `flutter gen-l10n` e `git diff --exit-code -- lib/l10n/generated`;
- o arquivo gerado deve ser produzido pelo mesmo Flutter 3.44.8 pinado na baseline;
- se upstream decidir futuramente não versionar generated code, essa política só muda em um PR arquitetural dedicado, alterando `.gitignore`, imports/build e CI em conjunto; não misturar os dois modelos.

## 3.9 Web locale bridge

Criar um adaptador Web pequeno, sem acoplar domínio a `dart:html`:

- atualizar `document.documentElement.lang` para BCP-47 efetivo;
- atualizar `document.documentElement.dir` para `ltr`/`rtl`;
- executar no bootstrap e a cada troca de App language;
- manter `index.html` com fallback canônico inicial;
- não tentar reescrever dados de catálogo ou metadata externa no DOM.

Teste Web deve inspecionar `lang` e `dir` após mudança de locale.

## 3.10 Rich text, InlineSpan e custom painting

Localização não pode presumir que todo texto user-facing nasce em `Text(String)`.

O scanner e a arquitetura devem cobrir:

- `Text.rich`;
- `RichText`;
- `TextSpan`/`InlineSpan`;
- `TextPainter`;
- `MarkdownBody`/`Markdown` e outros renderers de conteúdo rico;
- `WidgetSpan` e spans com gestures;
- texto desenhado por `CustomPainter`;
- labels passados como parâmetro para painters/idents;
- strings calculadas que chegam a esses sinks.

Para frases com partes estilizadas/clicáveis, não concatenar fragmentos independentes como:

    TextSpan(text: l10n.prefix)
    TextSpan(text: brand)
    TextSpan(text: l10n.suffix)

quando a gramática pode exigir outra ordem.

Preferir uma mensagem semanticamente completa com placeholder(s) e um renderer de spans reorder-safe. O contrato deve preservar:

- ordem definida pelo locale;
- estilos/gestures associados ao placeholder semântico;
- Semantics equivalente à frase completa;
- bidi isolation quando o placeholder contém dado externo.

Launch idents/custom painted copy devem ser classificados individualmente em:

- `LOCALIZE_STATIC`;
- `BRAND_ART_DIRECTION` com justificativa;
- `TECHNICAL_TOKEN`.

“Está desenhado no canvas” nunca é razão suficiente para sair do inventário.

## 3.11 Markdown, release notes e conteúdo rico remoto

`MarkdownBody(data: ...)` é sink de UI mesmo quando o texto vem de rede.

Casos confirmados:

    lib/main.dart
    lib/screens/settings_screen.dart

com `release.body`.

Política:

- chrome ao redor do Markdown usa ARB;
- fallback local como “Release notes will appear here...” usa ARB;
- body oficial recebe classificação `OFFICIAL_PRODUCT_CONTENT`;
- links dentro de Markdown mantêm URL como dado e localized accessible label quando o app fornecer label próprio;
- sanitize/security continua independente de i18n;
- não traduzir Markdown arbitrário de terceiros automaticamente;
- se release notes fizerem parte da experiência localizada, publicar variante pt-BR por contrato editorial/endpoint.


---

# 4. Refatorações obrigatórias descobertas pela reauditoria

## 4.1 SettingsRows

Problema atual:

SettingsRowContent armazena String title/subtitle e SettingsRows usa muitos static const com inglês.

Arquitetura alvo:

- metadata estrutural continua const: id, icon, url, capabilities;
- copy sai do objeto estático;
- title/subtitle são obtidos em build a partir de AppLocalizations;
- valores dinâmicos permanecem parâmetros.

Modelo sugerido:

    enum SettingsRowId {
      metadata,
      homePage,
      collections,
      playback,
      profiles,
      ...
    }

    class SettingsRowSpec {
      final SettingsRowId id;
      final IconData icon;
      final String? url;
    }

    LocalizedSettingsRow settingsRowCopy(
      AppLocalizations l10n,
      SettingsRowId id,
    )

Nenhuma tela deve voltar a duplicar título/subtítulo para contornar o mapper.

## 4.2 Settings Search

O índice deve ser locale-aware.

Cada entrada deve manter identidade estável:

- id;
- destination/action;
- icon;
- destructive/toggle metadata.

A copy de busca deve ser gerada no locale atual:

- localized title;
- localized subtitle;
- localized category;
- localized search keywords/synonyms.

O índice atual possui centenas de aliases ingleses manuais em `settings_screen.dart`. Não traduzir tudo mecanicamente.

Classificar aliases em:

- `LOCALIZED_ALIAS` — conceito natural ao usuário, ganha equivalente PT-BR;
- `TECHNICAL_ALIAS` — 4K, EPG, codec, API, URL etc., permanece canônico;
- `BRAND_ALIAS` — TorBox, Trakt, Simkl, WebDAV etc.;
- `LEGACY_EN_ALIAS` — termo inglês útil mantido deliberadamente para não regredir busca de power users.

Quando o locale muda, o índice é reconstruído/invalidado.

Não anexar a palavra inglesa `settings` universalmente. Usar keyword localizada do domínio de configurações e, se desejado por compatibilidade, manter `settings` como `LEGACY_EN_ALIAS` em PT-BR.

Como `_buildSearchIndex()` já é construído fresh ao abrir a busca, preservar essa propriedade; não introduzir cache global de strings localizadas sem invalidation.

## 4.3 Normalização de busca

Criar uma função única, testada, para pesquisa de UI, por exemplo `SettingsSearchNormalizer`.

Ela deve definir explicitamente:

- lower/case folding;
- trim;
- whitespace normalization;
- tratamento de acentos/diacríticos para pesquisa amigável;
- equivalência esperada para PT-BR;
- comportamento de caracteres não latinos.

Dart core não fornece normalização Unicode completa como um contrato pronto de produto; portanto não espalhar maps/regex ad-hoc por widgets. Centralizar a regra e os testes.

Aceite PT-BR mínimo:

    configuração == configuracao
    áudio == audio
    vídeo == video
    conexão == conexao
    reprodução == reproducao
    configurações == configuracoes

Também testar ç/Ç, ã/õ, ê/ô, múltiplos espaços e query vazia.

Para futuros idiomas, não reaproveitar cegamente remoção de diacríticos se isso puder mudar significado no idioma alvo.

Termos ingleses são preservados apenas como aliases deliberados (technical/brand/legacy), não por acidente.

Não usar normalização de busca para alterar dados persistidos.

## 4.4 Auditoria de acoplamento semântico

Criar gate que procure padrões de risco:

- comparação de title/label/subtitle com literal;
- switch em strings de display;
- persistência de label;
- IDs derivados de copy;
- lógica baseada em AppLocalizations retornado.

Primeiro caso confirmado a ser corrigido: All sources em source_sheet.dart.

## 4.5 Strings produzidas fora da camada visual

Inventariar funções em:

- utils;
- services;
- models;
- controllers;

que retornam sentenças destinadas à UI.

Migrar para:

A) valor/estado tipado + mapper localizado; ou  
B) formatter de apresentação localizado explicitamente.

Nunca mover AppLocalizations para services apenas para fazer o problema desaparecer.

## 4.6 Transformações de case

Classificar cada toUpperCase/toLowerCase como:

- protocol/storage/HTTP: não localizar;
- parsing técnico: não localizar;
- marca/token técnico: manter;
- copy localizada: preferir variante de tradução correta no ARB;
- dado externo user-facing: não alterar por razão de idioma, salvo decisão visual explícita e segura;
- teclado/input: tratar como requisito específico de escrita.

Evitar uppercase como mecanismo universal de estilo para idiomas futuros.

## 4.7 SourceSheet — hotspot confirmado

Tratar `lib/screens/video_player/widgets/source_sheet.dart` como migração atômica.

Obrigatório:

- identidade de grupo por enum/id, nunca `label == 'All sources'`;
- `All sources` apenas como copy localizada;
- `N source(s)` via ICU plural;
- `Playing` Semantics localizado;
- estados Fetching/failed/empty via ARB;
- file size pelo formatter locale-aware comum;
- não aplicar `toUpperCase()` indiscriminadamente à tradução;
- inicial/logomarca de addon permanece dado externo;
- badges técnicos `DIRECT/TORRENT/EXTERNAL` são classificados deliberadamente como tokens antes de manter uppercase.

## 4.8 Serviços Android — remover copy como estado

Antes de localizar notifications, refatorar:

    MediaStoreDownloadService
    LiveRecordingService
    RecordingAlarmReceiver

para trabalhar com estados tipados, por exemplo:

    enum class DownloadUiState {
      preparing, retrying, inProgress, paused, stopped, complete, failed
    }

    enum class RecordingUiState {
      starting, reconnecting, inProgress, complete, failed
    }

Proibido:

- `title == "Download complete"`;
- `title.startsWith("Starting")`;
- `title.startsWith("Saved")`;
- qualquer branch dependente de frase destinada ao usuário.

Estado -> Android string resource ocorre somente na borda de apresentação/notificação.

## 4.9 SupportRemoteConfig e copy oficial remota

Hotspot confirmado:

    assets/config/app_remote_config.json
    lib/services/support_remote_config_service.dart
    lib/main.dart
    lib/screens/settings_screen.dart

Hoje existem strings oficiais inglesas fora do ARB, inclusive:

- Support Debrify;
- Help fund development with a donation;
- Sponsor / Donate on Ko-fi;
- title/message de campanha remota.

Obrigatório:

1. separar dados remotos de copy local estável;
2. mover labels estáveis de Settings para AppLocalizations;
3. versionar schema de campanha localizada;
4. validar BCP-47 e fallback;
5. não persistir somente a variante já localizada se o cache for compartilhado entre locales;
6. invalidar/re-resolver apresentação quando App language muda;
7. scanner/inventory classifica payload fallback e consumers;
8. teste de offline/cached payload em en e pt-BR.

## 4.10 Catálogo remoto de engines/onboarding

O catálogo oficial de engines pode fornecer `display_name`/`description` usados na UI.

Política V4:

- identidade técnica do engine permanece canônica;
- nomes de marca podem permanecer dados;
- descrição editorial oficial precisa de schema locale-aware ou mapper local;
- engine importado por usuário/terceiro permanece `THIRD_PARTY_EXTERNAL_DATA`;
- ausência de PT-BR em catálogo oficial entra no relatório de completeness;
- onboarding nunca traduz texto arbitrário de engine de terceiros por IA/runtime automaticamente.

## 4.11 Notifications via background_downloader fora do Android nativo

`DownloadService` configura `TaskNotification` em plataformas não-Android com copy inglesa.

Requisitos:

- localizar running/complete/error/paused onde esse plugin realmente apresenta notifications;
- manter `{filename}` como dado/placeholder, nunca traduzi-lo;
- resolver copy antes de registrar/configurar notification;
- definir comportamento quando a tarefa sobrevive a troca de locale/restart;
- não depender de `BuildContext`;
- incluir esse caminho no scanner, testes e Definition of Done.

---

# 5. Contrato ARB

## 5.1 Template

app_en.arb é o contrato canônico.

Todo shipping locale deve possuir 100% das mensagens exigidas.

Cada arquivo ARB inclui `@@locale` coerente com o nome do arquivo. A CI valida que `app_pt_BR.arb` declara `pt_BR` e que a camada de persistência converte isso para `pt-BR` quando necessário.

## 5.2 Chaves semânticas

Padrão:

    <domain><screen/component><element/action/state>

Exemplos:

- settingsLanguageTitle
- settingsSearchHint
- settingsSearchNoResults
- profileEditSaveButton
- playerAudioTracksTitle
- playerSourcesAll
- downloadsDeleteConfirmTitle
- syncDeviceLastSeen
- errorNetworkUnavailable

Evitar chaves genéricas quando o contexto muda sentido.

## 5.3 Metadata obrigatória

Cada key possui @key com description.

Placeholder deve declarar:

- nome estável;
- type;
- example;
- descrição quando necessário.

CI falha se metadata obrigatória estiver ausente.

## 5.4 ICU

Usar plural/select para frases completas.

Nunca concatenar fragments traduzidos para montar sentenças.

Cobrir pluralização de:

- source;
- addon;
- episode;
- season;
- torrent;
- download;
- device;
- result;
- channel;
- item;
- minute;
- second;
- file;
- watched count;
- search count.

Testar pelo menos `0`, `1`, `2` e valores grandes em todos os plurais relevantes. Não assumir que futuro locale terá somente `one/other`.

## 5.5 Copy e termos técnicos

Manter sem tradução quando forem identidade/tokens:

- Debrify;
- Stremio;
- Trakt;
- Simkl;
- TorBox;
- Real-Debrid;
- AllDebrid;
- Premiumize;
- PikPak;
- WebDAV;
- codec names;
- WEB-DL;
- REMUX;
- HDR;
- Dolby Vision/DV quando usado como token;
- 2160p/1080p;
- URLs;
- MIME types.

Textos explicativos em torno deles são localizados.

## 5.6 Erros

Padronizar:

    UiErrorReason -> AppLocalizations

Categorias mínimas:

- networkUnavailable;
- timeout;
- authenticationExpired;
- permissionDenied;
- resourceUnavailable;
- malformedResponse;
- storageUnavailable;
- playbackUnavailable;
- unknown.

Detalhes técnicos ficam em logs/diagnostics com redaction apropriada.

## 5.7 Copy remota oficial

ARB continua sendo o contrato canônico para copy empacotada, mas não é a única fonte de copy própria.

Todo endpoint/config oficial capaz de renderizar texto deve declarar:

- schema/version;
- campos que são identidade/dados;
- campos que são localizáveis;
- locale tags em BCP-47;
- fallback;
- cache semantics;
- comportamento offline;
- ownership/review.

Para shipping locale, “caiu para en” é runtime-safe, porém não satisfaz completeness.

## 5.8 Nomes de idiomas

O catálogo de locales define autônimo, BCP-47 e direção independentemente do ARB ativo.

Exemplo conceitual:

    AppLocaleSpec(
      tag: 'pt-BR',
      nativeName: 'Português (Brasil)',
      direction: TextDirection.ltr,
    )

Não construir nomes de idiomas com `Locale.toString()` nem depender do locale atual para tornar o destino reconhecível.

---

# 6. Formatação e internacionalização de dados apresentados

## 6.1 Datas e horas

Remover DateFormat fixo para UI geral.

Usar locale ativo e decidir conscientemente se o timestamp deve ser:

- convertido para timezone local;
- mantido em UTC por ser diagnóstico;
- apresentado apenas como data;
- apresentado com hora.

Não aplicar locale de UI a formatos de protocolo/log.

## 6.2 12h/24h

Quando a plataforma oferece preferência do usuário, respeitá-la para UI.

Não deduzir 12h/24h apenas do idioma se a plataforma possui uma configuração independente.

## 6.3 Números

Usar NumberFormat para:

- contagens formatadas;
- percentuais;
- velocidades;
- ratings;
- estatísticas;
- números decimais de file size.

## 6.4 File size

Unidade técnica pode permanecer B/KB/MB/GB, mas decimal respeita locale.

Exemplo PT-BR:

    1,5 GB

A migração i18n não pode alterar silenciosamente a semântica 1024-base existente. Se o projeto quiser corrigir KB/MB/GB para KiB/MiB/GiB ou adotar SI 1000-base, isso é uma decisão funcional separada, com teste e release note.

## 6.5 Duração e tempo relativo

Centralizar:

- seconds/minutes/hours;
- x min ago;
- today;
- yesterday;
- last seen;
- updated ago;
- remaining;
- elapsed.

Frases usam ICU.

## 6.6 Ordenação e collation

Auditar listas A-Z/Z-A.

Não permitir que mudar App language altere silenciosamente tokens persistidos de sort.

Para labels internas localizadas, ordenação visual pode usar a representação localizada.

Para títulos de mídia e dados externos, manter a semântica de ordenação definida pelo recurso e não inventar tradução/collation sem requisito.

## 6.7 Todos os formatters user-facing entram no inventário

Não basta substituir `lib/utils/formatters.dart`.

A V3 confirmou chamadas diretas de `DateFormat` em outras superfícies, incluindo `main.dart`, Settings e Sync/Migrate. O scanner deve inventariar:

- `DateFormat`;
- `NumberFormat`;
- `String.format`/formatting nativo de número user-facing;
- `toStringAsFixed` user-facing;
- concatenação manual de unidade/percentual/tempo;
- pluralização manual.

Formatos de log/protocolo, como timestamp ISO UTC de diagnóstico, permanecem técnicos e não devem ser localizados.

### Regra de autoridade para intl

`MaterialApp.locale` não configura automaticamente todo `DateFormat`/intl já existente.

Para UI:

- formatter recebe `Locale`/BCP-47 efetivo explicitamente;
- preferir `Localizations.localeOf(context)` na borda de apresentação e passar o valor ao formatter;
- evitar `DateFormat(...)` user-facing sem locale;
- não usar `Intl.defaultLocale` como segunda autoridade global do produto;
- scanner/CI detecta novos `DateFormat`/formatters user-facing sem locale explícito;
- testes de override devem provar: system en + App language pt-BR gera data/número PT-BR e o inverso também.

---

# 7. RTL, layout, fontes, acessibilidade e input

Mesmo que PT-BR seja o primeiro locale, a infraestrutura deve evitar bloquear idiomas futuros.

## 7.1 Layout direcional

Auditar e migrar quando semântico:

- EdgeInsets.left/right -> EdgeInsetsDirectional.start/end;
- Alignment.centerLeft/right -> AlignmentDirectional;
- Positioned left/right quando representam início/fim;
- BorderRadius/ícones direcionais quando necessário;
- seta forward/back;
- rail/sidebar;
- D-pad traversal.

Nem todo left/right é semântico. Controles de mídia, timeline e geometria absoluta podem precisar permanecer físicos.

## 7.2 Pseudo-locales

Criar somente para debug/test:

- pseudo-LTR com expansão aproximada de 30–40%;
- pseudo-RTL.

Eles nunca entram em ShippingLocales.

## 7.3 Overflow

Testar:

- TV 720p/1080p;
- phone narrow width;
- tablet;
- desktop;
- textScale alto;
- titles longos;
- dialogs;
- bottom sheets;
- overlays;
- player;
- settings rows;
- chips;
- buttons;
- nav labels.

Proibir reduzir fonte agressivamente como solução padrão para tradução longa.

## 7.4 Font coverage

Verificar glyph coverage das fontes empacotadas.

PT-BR deve testar:

- á;
- à;
- â;
- ã;
- é;
- ê;
- í;
- ó;
- ô;
- õ;
- ú;
- ç;
- maiúsculas correspondentes.

Para idiomas futuros, definir fallback de fonte antes de declará-los shipping.

## 7.5 Semantics

Localizar:

- semanticLabel;
- button descriptions;
- tooltips;
- contentDescription nativo;
- status de progresso;
- toggles;
- image semantics quando realmente necessárias.

## 7.6 Teclado Debrify TV

O teclado próprio não pode ser considerado universalmente internacionalizado apenas porque a UI está traduzida.

Para PT-BR, definir teste de digitação/pesquisa com acentos.

Para novos scripts no futuro, escolher por locale entre:

- layout específico;
- teclado do sistema;
- IME/voice;
- suporte explícito no Debrify keyboard.

Não criar dezenas de layouts antes da necessidade, mas documentar a limitação para não declarar suporte falso.

Para PT-BR, o aceite não é apenas “buscar sem acento”. O usuário de TV deve conseguir inserir caracteres portugueses por pelo menos um caminho suportado: teclado próprio com acentos, IME do sistema ou voz. A escolha precisa ser documentada.

## 7.7 Bidi isolation

Para futuros locales RTL, interpolar dados externos com isolamento direcional quando necessário:

- URLs;
- paths;
- filenames;
- torrent/release names;
- usernames;
- códigos/IDs.

Não inserir marcas bidi em dados persistidos. O isolamento pertence à apresentação.

Testes devem cobrir copy RTL contendo dados LTR e copy LTR contendo dados RTL.

---

# 8. Plataformas nativas

# 8.1 Android / Android TV

## Recursos

Usar:

    android/app/src/main/res/values/strings.xml
    android/app/src/main/res/values-pt-rBR/strings.xml

Usar plurals/resources nativos quando a UI é nativa.

Auditar Kotlin e Java por:

- TextView.setText literal;
- contentDescription;
- Toast;
- dialog title/message/button;
- accessibility;
- startup/recovery;
- player;
- source browser;
- EPG/guide;
- DVR;
- notifications;
- foreground services;
- permission explanation;
- errors;
- track selectors;
- hints.

Cobrir no mínimo:

- AndroidTvTorrentPlayerActivity;
- TorboxTvPlayerActivity;
- MainActivity quando exibe copy;
- controllers/helpers nativos utilizados pelos players.

## Flutter -> native locale contract

O locale escolhido no Flutter deve ser transferido explicitamente para a camada nativa.

Contrato:

    uiLocale = system | BCP47

Centralizar em `NativeLocaleBridge` + `NativeLocaleStore`.

### NativeLocaleStore

A V3 **não cria uma segunda preferência/mirror** quando a mesma preferência device-owned pode ser lida por Dart e Android nativo.

Fonte persistida única:

    ui_locale_v1 = system | BCP47

Dart:

- lê/escreve por `DevicePreferences`;
- registra a key em `DevicePreferences.allowedKeys`;
- valida contra ShippingLocales antes da gravação.

Android nativo:

- `NativeLocaleStore` é um adapter de leitura da preferência física escrita pelo plugin;
- o projeto já lê `FlutterSharedPreferences` nativamente em outra integração; para a baseline atual, o adapter pode ler a key física correspondente a `ui_locale_v1` (por exemplo `flutter.ui_locale_v1`);
- esse detalhe fica isolado em um único adapter;
- contract test fixa o backing file/key da versão atual de `shared_preferences`;
- se o backend do plugin mudar, altera-se apenas o adapter/bridge.

Regras:

- nenhum segundo key `ui_locale_native_mirror_*`;
- Native nunca escreve/promove locale por conta própria;
- corrupção/ausência => system/en seguro;
- app/device reset limpa a mesma preferência junto com os demais SharedPreferences;
- nenhum profile id ou dado sensível nessa preferência.

`NativeLocaleBridge` sinaliza/aplica mudanças em superfícies nativas já vivas; ele não mantém outra fonte de persistência.

### Activities

Ao iniciar Activity nativa:

- informar/ler locale efetivo;
- aplicar Context/Configuration **antes de inflar views**;
- usar `LocaleList` em APIs modernas;
- preservar `layoutDirection`;
- evitar `Locale.setDefault` global como solução principal;
- não depender de label em inglês;
- testar que a Activity não perde playback/focus por mudança inesperada.

O manifest atual possui comportamento de `configChanges` diferente entre MainActivity e os players nativos; a implementação deve testar recreation/config change explicitamente, não presumir que todas Activities reagem igual.

### Services, Receivers e notifications

Cobrir pelo NativeLocaleStore:

- MediaStoreDownloadService;
- LiveRecordingService;
- RecordingAlarmReceiver;
- RecordingBootReceiver quando gerar/acionar UI indireta;
- qualquer novo foreground/background component.

Testar notification criada com Flutter morto e após reboot/process kill.

### Notification channels

- nome/description vêm de resources;
- criar/recriar idempotentemente o channel com copy do locale efetivo;
- testar channel existente após troca de idioma;
- nunca usar localized text como channel ID;
- ações Pause/Resume/Cancel/Stop e summaries usam resources/plurals.

## Android system per-app languages

Avaliar integração com o mecanismo de idioma por aplicativo do Android como melhoria de plataforma.

A adoção só deve ocorrer se os testes provarem que:

- Flutter e AppCompat permanecem sincronizados;
- Android TV não recria player em situação perigosa;
- preferência device-local continua consistente;
- restauração/backup não cria autoridade duplicada.

Se houver conflito, manter o AppLocaleController como autoridade do produto e usar o bridge explícito.

# 8.2 iOS

Cobrir:

- localizations declaradas no projeto Xcode;
- `knownRegions`/PBXVariantGroup atualizados;
- InfoPlist.strings;
- NSLocalNetworkUsageDescription;
- qualquer permission usage description adicional presente/futura;
- app display strings que realmente forem localizáveis;
- share/action extensions se surgirem;
- launch/recovery surfaces.

A baseline atual de iOS, tvOS e macOS possui `knownRegions = (en, Base)`; PT-BR precisa ser adicionado explicitamente aos três projetos e os arquivos localizados devem pertencer aos Variant Groups/targets corretos.

App Store só deve anunciar locale que esteja shipping.

**Limite de plataforma:** `InfoPlist.strings` e permission prompts são resolvidos pelo bundle/idioma do SO (incluindo per-app language do sistema), não pelo override Dart em runtime. Testar system en e system pt-BR separadamente; não prometer que mudar apenas o seletor interno reescreve um prompt já controlado pelo OS.

# 8.3 tvOS

Cobrir:

- Runner localization;
- InfoPlist.strings;
- TopShelf extension;
- textos nativos Swift;
- permission descriptions;
- player/remote hints nativos se existirem;
- focus engine com strings longas;
- Top Shelf metadata do app versus metadata externa;
- comportamento da extensão quando o app não está rodando;
- qualquer copy própria da extensão deve usar bundle localization ou snapshot localizado; títulos/sinopses externos continuam dados.

A extensão Top Shelf não pode depender de MethodChannel/Flutter ativo para descobrir idioma.

# 8.4 macOS

Cobrir:

- InfoPlist.strings;
- NSLocalNetworkUsageDescription;
- menus nativos/nib se possuírem copy user-facing;
- dialogs nativos;
- window chrome;
- permissões;
- `Base.lproj/MainMenu.xib` e seus menus padrão.

Para PT-BR, criar localização do menu (por exemplo `pt-BR.lproj/MainMenu.strings` ou mecanismo Xcode equivalente) preservando:

- selectors;
- key equivalents;
- systemMenu bindings;
- accessibility;
- APP_NAME/brand.

Não traduzir atalhos de teclado como se fossem copy.

# 8.5 Windows

Inventariar:

- runner resources;
- version info;
- window title;
- installer copy se versionada no repositório;
- dialogs nativos;
- protocol-handler UI.

Marca Debrify não precisa ser traduzida.

A auditoria V3 confirmou:

- `windows/installer.iss` possui somente idioma inglês;
- copy como `Create a desktop shortcut`, `Additional shortcuts` e `Launch Debrify` está hardcoded;
- `windows/runner/Runner.rc` ainda contém `com.example` em metadata template.

Requisitos:

- o workflow instala Inno Setup 6; adicionar no `[Languages]` a entrada PT-BR usando o language pack padrão dessa instalação, por exemplo:
  
      Name: "brazilianportuguese"; MessagesFile: "compiler:Languages\\BrazilianPortuguese.isl"

- o job Windows deve validar que esse arquivo existe no runner antes de chamar ISCC;
- localizar custom installer messages/tasks;
- manter nomes de protocolos/brand quando técnicos;
- validar instalador em sistema pt-BR e inglês;
- remover metadata `com.example` e usar metadata canônica do projeto;
- tratar correção de metadata como shell hygiene obrigatória da mesma fase, sem confundir campos técnicos com copy traduzível.

# 8.6 Linux

Inventariar runner GTK e recursos.

O window title debrify é marca e pode permanecer.

Qualquer mensagem nativa user-facing deve entrar no catálogo ou ser classificada/allowlisted.

# 8.7 Web

Corrigir primeiro a qualidade base:

- remover A new Flutter project do manifest e index.html;
- usar nome/descrição canônicos do Debrify;
- revisar title;
- revisar apple-mobile-web-app-title;
- revisar lang do documento quando viável;
- garantir que o shell não prometa idioma diferente do app;
- PWA metadata não pode ficar como placeholder de projeto Flutter.

O conteúdo dinâmico Flutter segue AppLocalizations.

Adicionar `WebLocaleBridge`:

- `<html lang>` possui fallback canônico inicial;
- em runtime, `document.documentElement.lang` recebe o BCP-47 efetivo;
- `document.documentElement.dir` acompanha a direção;
- atualização ocorre no bootstrap e a cada troca de idioma;
- teste browser verifica ambos.

Auditar também metadata visual ainda herdada do template Flutter, inclusive theme/background/orientation, sem alterá-la automaticamente se for decisão de produto não-i18n.

---

# 9. Inventário completo de superfícies

A auditoria de strings deve classificar cada ocorrência em uma tabela machine-readable com pelo menos:

- path;
- symbol/linha quando possível;
- plataforma;
- domínio;
- tipo de superfície;
- texto atual;
- classificação;
- key proposta;
- fase;
- status;
- justificativa se allowlisted.

Classificações:

- LOCALIZE_STATIC;
- LOCALIZE_ICU;
- LOCALIZE_NATIVE;
- EXTERNAL_DATA;
- USER_DATA;
- BRAND;
- TECHNICAL_TOKEN;
- PROTOCOL_IDENTIFIER;
- DEBUG_ONLY;
- TEST_ONLY;
- DEPRECATED;
- ALLOWLIST_WITH_REASON;
- SYSTEM_OWNED_UI;
- THIRD_PARTY_OWNED_UI;
- NATIVE_BACKGROUND_UI;
- GENERATED_CODE;
- REMOTE_PRODUCT_COPY;
- PRODUCT_CONTROLLED_REMOTE_CATALOG;
- RUNTIME_ASSET_COPY;
- BRAND_ART_DIRECTION.

O inventário deve incluir:

- Dart;
- Kotlin;
- Java;
- Swift;
- Objective-C se houver;
- XML;
- plist;
- storyboard/xib se aplicável;
- C/C++ runner;
- HTML;
- JSON de shell;
- notificações;
- contentDescription/Semantics;
- accessibility;
- permission descriptions;
- Text.rich/RichText/TextSpan/InlineSpan;
- TextPainter e copy desenhada em CustomPainter/Canvas;
- runtime-loaded JSON/YAML/CSV/Markdown e outros assets que alimentem UI;
- copy de remote config/endpoints oficiais;
- MarkdownBody/Markdown que renderizam conteúdo remoto/oficial;
- links/QR para guias oficiais acionados por fluxos do app;
- SVG text e inventário manual de raster/PDF runtime assets com potencial copy;
- notification copy configurada por plugins.

## 9.1 Escopo por reachability do artefato

A baseline V3 encontrou 1.574 arquivos nos roots principais de runtime auditados. Essa contagem é métrica de baseline, não número eterno.

O scanner deve:

1. incluir todo código próprio que chega aos artifacts publicados;
2. incluir `packages/` locais/vendorizados quando o package participa do runtime;
3. classificar exemplos/testes/dev/generated separadamente;
4. nunca excluir `packages/` ou `dev/` apenas por nome de pasta sem provar que não chegam ao artifact;
5. permitir allowlist de terceiro somente com path, reason, owner e revisão.

Arquivos recomendados:

    tool/l10n_inventory.json
    tool/l10n_allowlist.yaml

O inventário é determinístico e ordenado. A allowlist deve conter:

- rule/id;
- path/glob;
- literal/pattern quando aplicável;
- classification;
- reason;
- owner;
- review date/expiry opcional.

CI falha quando uma exceção allowlisted deixa de existir (allowlist stale), evitando acumular buracos permanentes.

---

# 10. Plano de migração

Cada fase só fecha quando os seus gates específicos passam.

## Fase 0 — Inventário, branch strategy e guardrails

Entregas:

- inventário inicial;
- allowlist versionada;
- scanners;
- semantic-coupling audit;
- política de shipping locale;
- pseudo locales de teste;
- métricas baseline;
- preflight de dependências Flutter 3.44.8 / intl 0.20.2;
- inventário por reachability com contagem baseline do artifact;
- contrato de ownership OS/app/third-party;
- contrato de DevicePreferences/NativeLocaleStore single-store;
- prova de exclusão por ProfilePreferencePortability;
- auditoria de backup rules por plataforma;
- grafo de runtime assets/loaders (pubspec + rootBundle/loaders próprios);
- inventário de remote product copy e endpoints oficiais;
- inventário de sinks RichText/TextSpan/TextPainter/Canvas;
- baseline de hardcoded directionality com classificação semântica/física.

Aceite:

- CI consegue detectar regressão de hardcoded UI;
- CI consegue detectar label usado como identidade;
- baseline de exceções está documentada.

## Fase 1 — Fundação Flutter

Entregas:

- flutter_localizations;
- l10n.yaml;
- app_en.arb;
- AppLocalizations;
- BuildContext.l10n;
- AppLocaleController;
- locale codec/resolution;
- ShippingLocales;
- todos os app roots configurados;
- formatters foundation;
- upgrade explícito de intl para constraint compatível com 0.20.2;
- l10n.yaml sem synthetic-package;
- generated-code policy;
- WebLocaleBridge foundation quando Web estiver habilitado em teste;
- RemoteProductCopy contract/resolver;
- locale catalog com autônimos;
- helper/renderer reorder-safe para rich text localizado.

Durante esta fase o seletor PT-BR ainda pode ficar oculto.

## Fase 2 — Navigation, Profiles, Onboarding e Settings

Prioridade máxima para usuário não falante de inglês.

Inclui:

- mobile nav;
- desktop nav;
- TV sidebar;
- Profiles;
- PIN/lock/autolock;
- onboarding;
- initial setup;
- settings root;
- settings pages;
- SettingsRows refactor;
- Settings Search refactor;
- Settings Search localized keywords;
- diagnostics/action labels;
- tooltips;
- Semantics;
- Support/campaign UI e fallback local de remote config;
- launch/splash/ident copy classificada e localizada ou allowlisted como brand art.

Aceite especial:

- busca funciona em PT-BR sem acento;
- mudança de locale reconstrói índice;
- D-pad/focus permanece determinístico.

## Fase 3 — Home, Discover, Search, Details, See All e Collections

Localizar chrome do app, não dados de catálogo.

Cobrir:

- empty/error/loading states;
- headings;
- See all;
- filtros;
- menus;
- watch progress actions;
- Continue Watching;
- labels estruturais;
- collection editor;
- sorting/filter copy.

## Fase 4 — Addons, Sources, Engines e Filters

Cobrir:

- addons screens;
- source rows/sheets;
- pinned source UI;
- source picker;
- filter settings;
- engine settings;
- indexer managers;
- badges settings.

Obrigatório:

- remover lógica dependente de All sources;
- auditar labels usados como sentinel;
- manter codecs/qualities/tokens canônicos;
- migrar SourceSheet como hotspot atômico: ids, plurais, semantics, case e file size;
- fechar contrato locale-aware para descrições do catálogo oficial de engines exibidas no onboarding/addons.

## Fase 5 — Debrid, Cloud, Files e Downloads

Cobrir:

- Real-Debrid;
- TorBox;
- AllDebrid;
- Premiumize;
- PikPak;
- WebDAV;
- cloud;
- downloads;
- account state;
- bind/add dialogs;
- auth errors;
- destructive confirmation;
- folder picker chrome;
- notifications do background_downloader em plataformas não-Android onde forem apresentadas ao usuário.

Nomes reais de arquivos/pastas do usuário permanecem dados.

## Fase 6 — Flutter Player

Cobrir:

- player controls;
- tracks;
- source sheet;
- guide;
- dock;
- subtitle/audio;
- sync;
- skip;
- sleep timer;
- playback errors;
- next episode;
- reconnect/recovery;
- shuffle/random;
- accessibility.

Aceite TV:

- nenhuma string longa quebra safe area;
- foco não muda de destino por tradução;
- open overlay mantém navegação correta.

## Fase 7 — Android Native Players

Cobrir todos os recursos e literals relevantes de:

- AndroidTvTorrentPlayerActivity;
- TorboxTvPlayerActivity;
- source browser/controllers;
- native track picker;
- EPG;
- startup errors;
- DVR;
- recording dialogs;
- accessibility;
- notifications nativas relacionadas.

Implementar NativeLocaleBridge + NativeLocaleStore e matriz override x system.

Antes de traduzir notification copy:

- remover branch por frases em Download/Recording services;
- migrar estado para enums/reason codes;
- extrair literals de XML para resources;
- localizar Services/Receivers/notification actions/channels;
- testar cold-start sem Flutter.

## Fase 8 — IPTV, Debrify TV e Stremio TV

Cobrir:

- IPTV widgets/settings;
- playlists;
- guide;
- favorites;
- catch-up/start-over;
- EPG states;
- recordings;
- Debrify TV;
- Magic TV;
- Stremio TV;
- channel/source sheets.

EPG/channel names recebidos externamente permanecem dados.

## Fase 9 — Tracking, Calendar, Community e integrações

Cobrir:

- Trakt;
- Simkl;
- MDBList;
- YouTube;
- Reddit;
- Lemmy;
- watched/unwatched;
- scrobble states;
- list management;
- calendar.

Marcas permanecem marcas.

## Fase 10 — Sync, Backup, Remote, Migration e Recovery

Cobrir:

- WebDAV sync;
- Setup guide WebDAV + link/QR + locale negotiation do conteúdo oficial;
- local backup;
- restore;
- migrate;
- pairing;
- remote keyboard/control;
- device management;
- transfer;
- vault;
- startup failure;
- recovery;
- migration status.

Critério crítico:

nenhum problema de locale pode impedir restauração ou recuperação de dados.

## Fase 11 — Apple, Desktop e Web shells

Cobrir:

- iOS;
- tvOS;
- Top Shelf;
- macOS;
- Windows;
- Linux;
- Web/PWA;
- permission descriptions;
- native metadata user-facing;
- macOS MainMenu;
- Windows Inno Setup pt-BR;
- Windows Runner.rc metadata hygiene;
- web placeholder cleanup;
- Web DOM lang/dir runtime sync;
- ownership/limites de locale OS-managed em Apple/installer.

## Fase 12 — Formatação, RTL, accessibility e input hardening

Executar auditoria transversal de:

- datas;
- horas;
- números;
- size;
- durations;
- relative time;
- case transform;
- RTL;
- text scale;
- Semantics;
- font coverage;
- TV keyboard;
- hardcoded TextDirection/Alignment/EdgeInsets/Positioned com classificação;
- RichText/TextSpan/TextPainter/custom painter copy;
- grammar/reordering de inline placeholders.

## Fase 13 — Zero-regression cleanup

Objetivo:

- hardcoded UI baseline -> zero não justificado;
- semantic coupling -> zero;
- shipping locale completeness -> 100%;
- placeholder parity -> 100%;
- native resource parity -> 100%;
- todos os allowlists com justificativa e owner.

## Fase 14 — Promoção PT-BR e release

Somente agora:

- adicionar pt_BR a ShippingLocales;
- habilitar App language no produto;
- validar fresh install;
- validar upgrade;
- validar system pt-BR;
- validar override inglês;
- validar override PT-BR;
- validar restart;
- validar profile switch;
- validar sync;
- validar player nativo;
- validar todos os artifacts;
- validar política de locale para release notes oficiais e guias acionados pelo app;
- publicar/registrar cobertura PRODUCT_EXPERIENCE_COMPLETENESS.

---

# 11. Quality gates de CI

## Gate 0 — dependency/toolchain preflight

Executar antes dos demais:

    flutter --version
    flutter pub get
    flutter pub deps
    flutter gen-l10n

Aceite:

- Flutter = 3.44.8 no baseline atual;
- `flutter_localizations` presente;
- `intl` resolve 0.20.2;
- nenhum `dependency_overrides` mascara intl;
- nenhum warning de `synthetic-package`;
- lockfile muda somente de forma explicável.

## Gate A — toolchain

Executar com a mesma versão de Flutter do upstream.

    flutter pub get
    flutter gen-l10n

Falhar se geração mudar arquivos inesperadamente.

Se generated code for versionado:

    git diff --exit-code -- lib/l10n/generated

## Gate B — ARB schema

Validar:

- JSON;
- key parity;
- @metadata;
- placeholder names;
- placeholder types;
- ICU;
- escape;
- plural/select;
- duplicate semantic keys;
- untranslated shipping messages.

## Gate C — ShippingLocales

Validar:

- shipping subset de generated locales;
- selector usa apenas shipping;
- en sempre disponível;
- pt-BR só após promoção formal.

## Gate D — Hardcoded UI

Scanner AST/token-aware preferencialmente, complementado por regex para plataformas nativas.

Detectar não apenas Text, mas:

- const String usado por UI;
- labels;
- titles;
- subtitles;
- hintText;
- helper return strings;
- status/error functions;
- SnackBar;
- Dialog;
- tooltip;
- Semantics;
- native setText/contentDescription;
- XML android:text/contentDescription/hint;
- NotificationCompat title/text/action/channel;
- Toast;
- platform menu copy;
- installer copy;
- plist usage descriptions;
- web metadata;
- Semantics labels/tooltip e qualquer API de announcement user-facing;
- Text.rich/RichText/TextSpan/InlineSpan;
- TextPainter/CustomPainter/Canvas text;
- TaskNotification/background_downloader e outros plugin notification builders;
- runtime asset copy em JSON/YAML/CSV/Markdown alcançável;
- remote product copy fallback/consumer paths;
- MarkdownBody/Markdown sources;
- official product links/QR/help destinations;
- visual runtime assets com texto embutido (SVG estrutural + revisão manual para raster/PDF).

## Gate E — Semantic coupling

Falhar em novos casos de:

- == 'display text';
- switch em copy;
- id derivado de label;
- persistence de localized string;
- lógica baseada em tradução.

## Gate F — Formatters

Testar ao menos:

- en;
- pt_BR;
- timezone;
- decimal;
- plural;
- date/time;
- relative time.

## Gate G — Flutter analysis/test

    flutter analyze
    flutter test

Durante migração podem existir suites segmentadas por fase, mas antes da promoção PT-BR a suite oficial completa deve estar verde ou qualquer falha preexistente deve estar registrada e reproduzida no baseline.

## Gate H — Android native

Executar unit/Robolectric/Gradle pertinentes.

Verificar:

- resource parity entre values e values-pt-rBR;
- zero literal user-facing em layouts XML fora de allowlist;
- ambas Activities de player;
- NativeLocaleStore;
- Services/Receivers com processo Flutter morto;
- notifications, actions, plurals e channels;
- ausência de branch baseado em localized/display text.

## Gate I — Platform build matrix

Antes do release, compilar os targets publicados pelo workflow atual e validar que l10n não quebra packaging:

- Android;
- macOS;
- Windows;
- iOS;
- tvOS;
- Linux x64;
- Linux arm64.

Web também deve compilar em CI quando a superfície Web fizer parte do suporte/publicação esperado, mesmo que não seja artifact do workflow de release atual.

## Gate J — Pseudo/layout

Widget/golden/manual:

- expanded LTR;
- pseudo RTL;
- text scale;
- TV focus;
- narrow phone;
- desktop.

## Gate K — No secrets/log regression

Mudança de erros/localização não deve colocar tokens, URLs sensíveis, credentials ou payloads privados em mensagens de UI/log.

## Gate L — Remote/runtime product copy

Falhar ou registrar blocker quando:

- campo user-facing oficialmente controlado pelo produto não possui política de locale;
- fallback asset/remote config contém copy própria fora do contrato;
- shipping locale cai para en em `REMOTE_PRODUCT_COPY`;
- catálogo remoto oficial exibe descrição editorial sem locale/mapeamento aprovado;
- cache de remote copy pode servir variante do locale anterior;
- runtime asset user-facing escapa do inventário.

O relatório de completeness deve separar:

    ARB
    native resources
    remote product copy
    product-controlled remote catalog
    runtime asset copy
    official product content
    runtime visual assets
    approved external/user data

## Gate M — Directionality and inline-text safety

Scanner estático deve reportar em runtime app-owned:

- `TextDirection.ltr/rtl` hardcoded;
- `Alignment.*Left/*Right`;
- `EdgeInsets` físicos quando start/end é semântico;
- `Positioned(left/right)` em UI direcional;
- directional icons/arrows;
- concatenação/ordem rígida de `TextSpan` para frases localizadas.

Cada finding precisa ser corrigido ou classificado como físico/intencional com allowlist justificada.

Teste de pseudo-RTL deve incluir rich text, dados LTR interpolados e medição de texto.


---

# 12. Matriz de testes obrigatórios

## Controller/resolution

- fresh install;
- system en;
- system pt-BR;
- system pt-PT;
- system pt sem região;
- en-US/en-GB fallback;
- multiple preferred locales;
- manual en;
- manual pt-BR;
- invalid persisted tag;
- pt_BR legacy tag;
- unknown language;
- system locale changes while followsSystem;
- system change ignored while manual override;
- restart;
- Web DOM lang/dir update;
- app locale vs OS-owned locale boundary.

## Isolation

- trocar idioma não altera metadata;
- não altera artwork;
- não altera audio;
- não altera subtitle;
- não altera region;
- não altera sort/filter tokens;
- não altera addon behavior;
- profile switch não altera locale;
- WebDAV não sobrescreve locale;
- restore/profile import não sobrescreve locale.

## Bootstrap/recovery

- startup failure em en/PT-BR;
- recovery em en/PT-BR;
- migration screen;
- Linux vault;
- preference read failure;
- corrupt storage.



## Remote/runtime copy

- fallback asset de SupportRemoteConfig em en/PT-BR;
- cached remote config após troca en -> pt-BR e pt-BR -> en;
- campaign exact locale/fallback/unknown locale;
- malformed/partial remote payload não bloqueia UI;
- labels estáveis de Settings vêm de ARB, não de payload inglês;
- catálogo oficial de engines apresenta descrição PT-BR quando shipping;
- engine de terceiro continua dado externo;
- offline mantém contrato previsível sem congelar locale anterior;
- completeness report acusa fallback inglês de copy oficial;
- release.body/Markdown tem ownership explícito e fallback chrome localizado;
- Setup guide WebDAV usa destino com locale policy sem exigir QR diferente por idioma;
- conteúdo oficial essencial acionado pelo app não é confundido com third-party data.

## Rich text / custom painting

- `Text.rich` e `RichText` aparecem corretamente em en/PT-BR;
- placeholders estilizados podem mudar de ordem por locale;
- Semantics produz frase completa, não fragmentos desconexos;
- pseudo-RTL não quebra spans nem links;
- launch idents têm classificação explícita;
- custom painted copy localizada recebe locale/direction quando aplicável;
- `catalog_item_detail_screen` e qualquer medição de texto app-owned não força LTR sem justificativa.


## Settings

- rows localizadas;
- dynamic subtitles;
- search title/subtitle/category;
- search accent folding;
- localized keywords;
- locale change com search aberto;
- D-pad focus;
- toggles.

## Flutter player

- controls;
- tracks;
- overlays;
- next episode;
- errors;
- sleep timer;
- source sheet;
- long labels;
- semantics.

## Android native

Matriz mínima:

1. system en + system mode;
2. system pt-BR + system mode;
3. system pt-BR + override en;
4. system en + override pt-BR.

Executar em:

- AndroidTvTorrentPlayerActivity;
- TorboxTvPlayerActivity.

Validar:

- resource selection;
- no crash;
- no playback restart indevido;
- focus;
- accessibility;
- dialogs/EPG/source picker;
- Service/Receiver notification após process kill;
- notification channel já existente após troca de locale;
- actions Pause/Resume/Cancel/Stop;
- plural 0/1/2 em summaries.

## Apple/platform

- localized permission descriptions;
- Xcode supported languages;
- tvOS Top Shelf não quebra;
- macOS permission copy;
- Web manifest/index sem placeholder;
- macOS menu pt-BR;
- Windows installer pt-BR;
- Windows Runner.rc sem com.example;
- Apple permission prompt testado por system/per-app language, sem assumir que override Dart controla InfoPlist.strings;
- tvOS Top Shelf sem dependência de Flutter ativo.

## Voice/input

- speech locale default não muda ao trocar App language;
- BCP-47 explícito chega a EXTRA_LANGUAGE somente quando política de input manda;
- PT-BR possui caminho real para inserir acentos no TV;
- denial/unavailable recognizer usa copy localizada sem bloquear teclado.

## Pseudo/RTL

- 30–40% expansão;
- mixed LTR/RTL data;
- bidi isolation para URL/path/filename/user data;
- numbers;
- D-pad;
- directional icons;
- ellipsis only where acceptable.

---

# 13. PT-BR — padrão editorial

Objetivo:

- natural para brasileiro;
- curto onde espaço é crítico;
- consistente entre Flutter e Android nativo;
- não traduzir marca;
- não traduzir jargão técnico quando tradução piorar clareza.

Glossário inicial continua em docs/GLOSSARIO_PT_BR.md.

Decisões editoriais devem ser documentadas, por exemplo:

- Addons: manter Addons enquanto a comunidade/produto usar o termo;
- Player: decidir de forma global entre Player e Reprodutor, não alternar;
- Sources: Fontes no contexto de playback/search;
- Settings: Configurações;
- Home: Início;
- Browse: Explorar;
- Search: Pesquisar;
- Continue Watching: Continuar assistindo;
- System default: Padrão do sistema.

Antes do release, executar revisão humana de toda copy PT-BR em contexto, não somente do ARB isolado.

---

# 14. Estratégia de implementação e PRs upstream

Evitar um PR monolítico.

Sequência recomendada:

1. PR A — dependency preflight (intl 0.20.2) + foundation + tests + ShippingLocales en-only + inventory contracts + runtime/remote copy ownership + rich-text/directionality guardrails.
2. PR B — SettingsRows/Settings Search + navigation/profiles/onboarding + SupportRemoteConfig fixed-copy split + language autonyms.
3. PR C — discover/search/details/collections.
4. PR D — addons/sources/filters/debrid/downloads + official engine-catalog localization + plugin download notifications.
5. PR E — Flutter player + TV surfaces.
6. PR F — Android native resources + NativeLocaleBridge + NativeLocaleStore + background notifications.
7. PR G — IPTV/Debrify TV/Stremio TV/tracking/sync/recovery.
8. PR H — Apple/Desktop/Web native surfaces + macOS menu + Windows installer/metadata + Web lang/dir + formatting/a11y hardening.
9. PR I — PT-BR completion, full audit, promotion to ShippingLocales.

Cada PR deve:

- ser bisectável;
- não alterar IDs/protocolos sem necessidade;
- incluir testes do domínio;
- reduzir ou manter a baseline de hardcoded strings, nunca aumentar;
- listar novas ARB keys;
- listar allowlist adicionada/removida;
- indicar risco de TV/player;
- ter rollback simples.

Se upstream preferir menos PRs, combinar domínios adjacentes sem perder gates.

---

# 15. Rollout e rollback

## 15.1 Feature visibility

Durante migração:

- infraestrutura pode existir;
- PT-BR pode ser testado internamente;
- seletor público fica indisponível enquanto a cobertura não for completa.

## 15.2 Rollback seguro

Se locale override causar regressão:

- manter en como fallback;
- permitir limpar ui_locale_v1;
- startup nunca depende de carregar tradução remota;
- ARBs são empacotados no app;
- falha do controller devolve system/en;
- native player recebe fallback seguro;
- `ui_locale_v1` ausente/corrompido retorna system/en;
- app/device reset não deixa locale nativo órfão;
- Services/Receivers continuam funcionais sem Flutter;
- OS-owned Apple/installer surfaces não são tratadas como falha do override interno.

## 15.3 Observabilidade

Registrar somente eventos técnicos não sensíveis, por exemplo:

- locale resolution fallback;
- invalid persisted locale;
- missing localization contract em debug/test.

Não registrar conteúdo digitado, nomes privados, tokens ou mensagens externas arbitrárias.

---

# 16. Riscos principais e mitigação

## Risco: gigantesco churn de ARB

Mitigação:

- migração por domínio;
- naming guide;
- pequenas revisões;
- sort/format automático.

## Risco: merge conflicts em Settings

Mitigação:

- refatorar registry primeiro;
- minimizar PRs long-lived;
- rebase frequente.

## Risco: TV focus/layout quebra com texto maior

Mitigação:

- pseudo locale;
- testes D-pad;
- directional layout;
- revisão em hardware/emulador.

## Risco: player nativo em idioma diferente do Flutter

Mitigação:

- NativeLocaleBridge;
- matriz system/override;
- resource tests.

## Risco: texto localizado vira dado persistido

Mitigação:

- semantic coupling gate;
- stable enums/ids;
- code review checklist.

## Risco: mixed-language release

Mitigação:

- ShippingLocales separado;
- PT-BR só promovido após 100%.

## Risco: strings fora de widgets escapam do scanner

Mitigação:

- inventory cross-layer;
- AST/static search;
- native scan;
- allowlist controlada.

## Risco: Android system language cria duas autoridades

Mitigação:

- uma autoridade declarada;
- integração Android apenas após prova de sincronização;
- testes de restore/recreation.

## Risco: background Android usa idioma antigo

Mitigação:

- uma única `ui_locale_v1` em DevicePreferences;
- NativeLocaleStore lê o mesmo backing store;
- NativeLocaleBridge apenas atualiza superfícies vivas;
- component-local Context;
- re-upsert de notification channels;
- contract test do backing key atual;
- testes com process kill/reboot/reset.

## Risco: Apple/installer não segue override interno

Mitigação:

- ownership matrix explícita;
- bundle localizations completas;
- teste por system/per-app language;
- UX/documentação sem prometer controle que a plataforma não oferece.



## Risco: remote config mantém inglês fora do ARB

Mitigação:

- ownership explícito de remote product copy;
- labels estáveis no ARB;
- payload multilíngue versionado para campanha;
- cache locale-safe;
- completeness gate acusa fallback inglês.

## Risco: scanner ignora rich text/assets/canvas

Mitigação:

- sinks TextSpan/TextPainter/CustomPainter no AST scan;
- asset graph a partir de pubspec/rootBundle/loaders;
- runtime asset/remote copy inventory;
- stale allowlist;
- screenshots/pseudo tests em superfícies críticas.

## Risco: hardcoded directionality passa despercebido

Mitigação:

- Gate M;
- classificação físico x semântico;
- pseudo-RTL;
- teste de rich text e text measurement.

## Risco: scan “100%” ignora package vendorizado

Mitigação:

- inventory por reachability;
- packages runtime em escopo;
- generated/test/dev classificados;
- stale allowlist falha CI.

---

# 17. Definition of Done — PT-BR

O trabalho está concluído somente quando TODOS os itens abaixo forem verdadeiros:

- [ ] v0.10.0-beta.1/commit alvo revalidado ou baseline atualizada.
- [ ] flutter_localizations configurado.
- [ ] intl atualizado e resolvendo 0.20.2 sob Flutter 3.44.8.
- [ ] l10n.yaml sem synthetic-package deprecated.
- [ ] gen_l10n reproduzível.
- [ ] app_en.arb é template.
- [ ] app_pt_BR.arb possui 100% das keys exigidas.
- [ ] 100% dos placeholders são compatíveis.
- [ ] ICU válido.
- [ ] PT-BR promovido explicitamente para ShippingLocales.
- [ ] App language possui System default, English e Português (Brasil).
- [ ] Preferência é device-local.
- [ ] Profile/WebDAV não alteram App language.
- [ ] Todos os roots Flutter localizados.
- [ ] SettingsRows não guarda copy inglesa como identidade.
- [ ] Settings Search é localizada e accent-friendly.
- [ ] Zero lógica depende de display strings, inclusive Download/Recording services nativos.
- [ ] Zero hardcoded UI conhecido fora de allowlist justificada.
- [ ] SupportRemoteConfig não injeta fixed product copy inglesa fora do ARB.
- [ ] Campaign/remote product copy possui schema locale-aware e fallback testado.
- [ ] Catálogo oficial de engines/onboarding possui política locale-aware para copy editorial.
- [ ] Runtime-loaded assets user-facing estão no inventory/completeness report.
- [ ] Notifications via background_downloader estão localizadas onde user-facing.
- [ ] Text.rich/RichText/TextSpan/TextPainter/CustomPainter user-facing estão localizados ou classificados.
- [ ] Hardcoded directionality app-owned possui zero findings não classificados.
- [ ] Language picker usa autônimos estáveis.
- [ ] MarkdownBody/Markdown user-facing possui source ownership/classification.
- [ ] Release notes oficiais possuem política editorial de locale registrada.
- [ ] WebDAV Setup guide/link/QR possui destino com política de locale.
- [ ] Runtime visual assets com potencial texto foram auditados/classificados.
- [ ] Formatters são locale-aware onde aplicável.
- [ ] Android native values-pt-rBR completo.
- [ ] Zero android:text/contentDescription literal user-facing fora de allowlist.
- [ ] ui_locale_v1 pertence a DevicePreferences.allowedKeys.
- [ ] ProfilePreferencePortability rejeita ui_locale_v1.
- [ ] Nenhum raw SharedPreferences access novo foi introduzido para o controller.
- [ ] NativeLocaleStore lê a mesma preferência canônica, sem mirror duplicado.
- [ ] NativeLocaleStore cobre cold-start de Service/Receiver.
- [ ] Notifications/actions/channels localizados e testados após process kill.
- [ ] Ambas Activities nativas principais do player testadas.
- [ ] NativeLocaleBridge testado.
- [ ] iOS InfoPlist strings/localizations revisados.
- [ ] iOS/tvOS/macOS knownRegions incluem PT-BR e Variant Groups/target membership estão corretos.
- [ ] tvOS Runner/Top Shelf revisados.
- [ ] macOS InfoPlist strings/localizations revisados.
- [ ] macOS MainMenu localizado para PT-BR.
- [ ] Windows/Linux native surfaces auditadas.
- [ ] Windows installer possui PT-BR e custom copy localizada.
- [ ] Windows Runner.rc não contém metadata template com.example.
- [ ] Web manifest/index sem placeholder Flutter.
- [ ] Web documentElement lang/dir acompanha locale efetivo.
- [ ] Semantics/contentDescription localizados.
- [ ] PT-BR font/glyph coverage validada.
- [ ] Pseudo-LTR passa em superfícies críticas.
- [ ] Pseudo-RTL não revela acoplamento físico evitável.
- [ ] textScale/overflow revisado.
- [ ] TV D-pad/focus passa.
- [ ] TV possui caminho suportado para digitar caracteres PT-BR.
- [ ] Speech/input locale não é acoplado implicitamente ao UI locale.
- [ ] Bidi isolation testado para dados externos em pseudo-RTL.
- [ ] flutter analyze passa conforme baseline acordada.
- [ ] suites l10n passam.
- [ ] suite completa passa ou diferenças preexistentes estão provadas.
- [ ] Android native tests passam.
- [ ] build matrix dos artifacts publicados passa.
- [ ] revisão humana PT-BR em contexto concluída.
- [ ] README/documentação atualizados.
- [ ] rollback path testado.

---

# 18. Checklist de auditoria para idiomas futuros

Antes de adicionar qualquer novo locale:

1. confirmar font/script support;
2. adicionar ARB completo;
3. definir autônimo do idioma;
4. definir política de script/region;
5. revisar plurais;
6. revisar formatos;
7. revisar case rules;
8. revisar input/TV keyboard;
9. revisar RTL se aplicável;
10. revisar platform-native resources;
11. executar pseudo/layout tests;
12. executar human review;
13. somente então adicionar a ShippingLocales.

---

# 19. Critério de completude da auditoria

Nenhuma auditoria estática isolada consegue provar literalmente que não existe uma única string construída dinamicamente em runtime em todos os caminhos. Por isso, a completude deste plano não depende de promessa subjetiva de 100%.

Ela é transformada em evidência mensurável por:

- inventário versionado;
- scanners;
- semantic-coupling gate;
- key/resource parity;
- shipping locale gate;
- testes de runtime;
- platform build matrix;
- pseudo-locale;
- revisão manual em superfícies críticas;
- baseline de exceções com justificativa.

Esse conjunto é o mecanismo que permite chegar a uma cobertura demonstrável e sustentável, inclusive depois que o Debrify continuar evoluindo.

---

# 20. Ordem recomendada para começar

1. Não iniciar tradução massiva ainda.
2. Implementar Fase 0.
3. Implementar Fase 1 com ShippingLocales = en.
4. Refatorar SettingsRows e Settings Search antes de traduzir Settings.
5. Criar inventário e eliminar semantic coupling.
6. Migrar por domínio.
7. Integrar Android native locale.
8. Fechar Apple/Desktop/Web.
9. Rodar hardening.
10. Promover pt-BR somente no último gate.

Este é o caminho que minimiza retrabalho, evita UI parcialmente traduzida e mantém o Debrify funcional durante toda a migração.

---

# 21. Evidência mínima que acompanha cada implementação

Nenhuma fase é considerada concluída somente por “funcionou no meu dispositivo”.

Cada PR deve anexar, conforme aplicável:

- commit upstream rebaseado;
- output de `flutter --version`;
- lock/resolução de dependências;
- diff de inventory/allowlist;
- contagem de hardcoded findings antes/depois;
- ARB/resource parity report;
- testes unit/widget/native;
- screenshots ou gravação das superfícies críticas en/PT-BR;
- prova de TV focus quando a mudança toca TV;
- prova de process-kill para background Android;
- build artifact correspondente;
- limitações OS-owned registradas.

A meta da V4 é transformar “100%” de uma promessa subjetiva em um conjunto auditável de provas reproduzíveis que cobre source code, plataformas nativas, assets de runtime, copy oficial remota e conteúdo oficial acionado pelo produto.
