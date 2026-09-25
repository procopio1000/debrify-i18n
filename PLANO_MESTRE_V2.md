# Plano Mestre V2 — Internacionalização (i18n/l10n) completa do Debrify

**Status:** versão consolidada, re-auditada e implementation-ready  
**Projeto de planejamento:** procopio1000/debrify-i18n  
**Projeto-alvo upstream:** varunsalian/debrify  
**Baseline verificada:** v0.10.0-beta.1  
**Commit-alvo:** 9619c10b06ee919cacbe996b30be7739dc09c6d6  
**Data da reauditoria:** 2026-09-25  
**Primeiro locale de release:** Português do Brasil (pt-BR)  
**Template canônico:** inglês (en)  
**Escopo:** Flutter + Android nativo/TV + iOS + tvOS + macOS + Windows + Linux + Web + acessibilidade + formatação + busca + CI + release

---

# 0. Resultado da reauditoria do V1

O V1 estava na direção correta, mas não estava completo o suficiente para ser tratado como plano final.

## 0.1 Falha estrutural encontrada no próprio V1

O arquivo PLANO_MESTRE_V1.md termina no começo da Fase 10:

- Fase 10 — Plataformas não-Flutter
- Android
- values/strings.xml
- values-pt-rBR/strings.xml

Não há conclusão da Fase 10, não há fases finais de hardening/release, não há estratégia de rollout/upstream, não há Definition of Done consolidada e não há fechamento de riscos.

O V2 substitui o V1 como plano atual, mas o V1 deve permanecer no repositório como histórico.

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

## 2.5 Preferência de App language é device-local em V2

Política inicial:

- local ao dispositivo;
- disponível antes do ProfileGate;
- não pertence ao perfil;
- não é sincronizada por WebDAV;
- não é transportada automaticamente em backup de perfil;
- não muda ao trocar de perfil;
- pode ser resetada para System default;
- preferência inválida/corrompida nunca pode impedir startup.

---

# 3. Arquitetura Flutter alvo

## 3.1 Stack

Adicionar:

    flutter_localizations:
      sdk: flutter

Manter intl na versão compatível com o Flutter realmente usado pelo upstream.

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

Antes do merge, validar as flags com flutter gen-l10n --help usando exatamente o Flutter 3.44.8 do workflow.

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
- ler ui_locale_v1;
- null/system = seguir sistema;
- aceitar BCP 47;
- normalizar aliases legados;
- rejeitar locale não shipping;
- persistir atomicamente;
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

Persistência:

    ui_locale_v1 = system | en | pt-BR | ...

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

Política V2 inicial:

- en é fallback;
- pt-BR é explícito;
- pt-PT não deve ser rotulado como Português (Brasil);
- pt sem região precisa de política documentada e teste;
- valor legado pt_BR é normalizado para pt-BR;
- valor desconhecido não causa crash e volta a system/en.

## 3.7 Todos os roots Flutter devem compartilhar a mesma configuração

Cobrir MaterialApp de:

- app principal;
- startup failure;
- migration/update;
- recovery;
- Linux vault bootstrap;
- quaisquer hosts reais executáveis pelo usuário.

Criar helper compartilhado de delegates/locales/resolution para impedir drift.

Hosts de testes e ferramentas só precisam de localization quando renderizam widgets reais cuja copy depende dela.

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

Quando o locale muda, o índice é reconstruído.

Não anexar a palavra inglesa settings universalmente. Criar keyword localizada para o domínio de configurações.

## 4.3 Normalização de busca

Criar uma função única, testada, para pesquisa de UI.

Ela deve definir explicitamente:

- lower/case folding;
- trim;
- whitespace normalization;
- tratamento de acentos/diacríticos para pesquisa amigável;
- equivalência esperada para PT-BR;
- comportamento de caracteres não latinos.

Exemplo de aceite:

- configuracoes encontra Configurações;
- legenda encontra a entrada traduzida pertinente;
- termos ingleses podem ser preservados apenas como aliases deliberados quando forem termos técnicos conhecidos, não por acidente.

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

---

# 5. Contrato ARB

## 5.1 Template

app_en.arb é o contrato canônico.

Todo shipping locale deve possuir 100% das mensagens exigidas.

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

Centralizar em NativeLocaleBridge.

Ao iniciar Activity nativa:

- informar o locale efetivo/override;
- criar resources/context apropriado;
- não depender de label em inglês;
- testar que a Activity não perde playback/focus por mudança inesperada.

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
- InfoPlist.strings;
- NSLocalNetworkUsageDescription;
- qualquer permission usage description adicional presente/futura;
- app display strings que realmente forem localizáveis;
- share/action extensions se surgirem;
- launch/recovery surfaces.

App Store só deve anunciar locale que esteja shipping.

# 8.3 tvOS

Cobrir:

- Runner localization;
- InfoPlist.strings;
- TopShelf extension;
- textos nativos Swift;
- permission descriptions;
- player/remote hints nativos se existirem;
- focus engine com strings longas;
- Top Shelf metadata do app versus metadata externa.

# 8.4 macOS

Cobrir:

- InfoPlist.strings;
- NSLocalNetworkUsageDescription;
- menus nativos/nib se possuírem copy user-facing;
- dialogs nativos;
- window chrome;
- permissões.

# 8.5 Windows

Inventariar:

- runner resources;
- version info;
- window title;
- installer copy se versionada no repositório;
- dialogs nativos;
- protocol-handler UI.

Marca Debrify não precisa ser traduzida.

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
- ALLOWLIST_WITH_REASON.

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
- permission descriptions.

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
- métricas baseline.

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
- formatters foundation.

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
- Semantics.

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
- manter codecs/qualities/tokens canônicos.

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
- folder picker chrome.

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

Implementar NativeLocaleBridge e matriz override x system.

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
- web placeholder cleanup.

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
- TV keyboard.

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
- validar todos os artifacts.

---

# 11. Quality gates de CI

## Gate A — toolchain

Executar com a mesma versão de Flutter do upstream.

    flutter pub get
    flutter gen-l10n

Falhar se geração mudar arquivos inesperadamente.

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
- plist usage descriptions;
- web metadata.

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

Verificar resource parity entre values e values-pt-rBR.

Testar ambas Activities de player.

## Gate I — Platform build matrix

Antes do release, compilar os targets publicados pelo projeto e validar que l10n não quebra packaging.

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

---

# 12. Matriz de testes obrigatórios

## Controller/resolution

- fresh install;
- system en;
- system pt-BR;
- multiple preferred locales;
- manual en;
- manual pt-BR;
- invalid persisted tag;
- pt_BR legacy tag;
- unknown language;
- system locale changes while followsSystem;
- system change ignored while manual override;
- restart.

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
- dialogs/EPG/source picker.

## Apple/platform

- localized permission descriptions;
- Xcode supported languages;
- tvOS Top Shelf não quebra;
- macOS permission copy;
- Web manifest/index sem placeholder.

## Pseudo/RTL

- 30–40% expansão;
- mixed LTR/RTL data;
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

1. PR A — foundation + tests + ShippingLocales en-only.
2. PR B — SettingsRows/Settings Search + navigation/profiles/onboarding.
3. PR C — discover/search/details/collections.
4. PR D — addons/sources/filters/debrid/downloads.
5. PR E — Flutter player + TV surfaces.
6. PR F — Android native resources/NativeLocaleBridge.
7. PR G — IPTV/Debrify TV/Stremio TV/tracking/sync/recovery.
8. PR H — Apple/Desktop/Web native surfaces + formatting/a11y hardening.
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
- native player recebe fallback seguro.

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

---

# 17. Definition of Done — PT-BR

O trabalho está concluído somente quando TODOS os itens abaixo forem verdadeiros:

- [ ] v0.10.0-beta.1/commit alvo revalidado ou baseline atualizada.
- [ ] flutter_localizations configurado.
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
- [ ] Zero lógica depende de display strings.
- [ ] Zero hardcoded UI conhecido fora de allowlist justificada.
- [ ] Formatters são locale-aware onde aplicável.
- [ ] Android native values-pt-rBR completo.
- [ ] Ambas Activities nativas principais do player testadas.
- [ ] NativeLocaleBridge testado.
- [ ] iOS InfoPlist strings/localizations revisados.
- [ ] tvOS Runner/Top Shelf revisados.
- [ ] macOS InfoPlist strings/localizations revisados.
- [ ] Windows/Linux native surfaces auditadas.
- [ ] Web manifest/index sem placeholder Flutter.
- [ ] Semantics/contentDescription localizados.
- [ ] PT-BR font/glyph coverage validada.
- [ ] Pseudo-LTR passa em superfícies críticas.
- [ ] Pseudo-RTL não revela acoplamento físico evitável.
- [ ] textScale/overflow revisado.
- [ ] TV D-pad/focus passa.
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
