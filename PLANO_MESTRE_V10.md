# Plano Mestre V10 — Internacionalização (i18n/l10n) completa do Debrify

**Status:** versão consolidada, décima auditoria extrema, implementation-ready, toolchain-verified, packaging-verified-by-design, cross-runtime-verified, cross-device-semantic-verified, unicode/composition-hardened, temporal-semantics-hardened, async-locale-race-hardened, declarative-presentation-dataflow-hardened, coverage-closure-hardened, canonical-gate-registry e baseline-drift-hardened  
**Projeto de planejamento:** procopio1000/debrify-i18n  
**Projeto-alvo upstream:** varunsalian/debrify  
**Baseline verificada:** v0.10.0-beta.1  
**Commit-alvo:** 9619c10b06ee919cacbe996b30be7739dc09c6d6  
**Data da reauditoria:** 2026-09-26  
**Primeiro locale de release:** Português do Brasil (pt-BR)  
**Template canônico:** inglês (en)  
**Escopo:** Flutter + Android nativo/TV + Services/Receivers + iOS + tvOS + macOS + Windows + Linux + Web + packages runtime + acessibilidade + formatação + busca + CI + release  
**Escopo estrutural revalidado na V8, V9 e novamente na V10 pelo Git tree do commit-alvo:** 3.199 arquivos/blobs no repositório; 1.736 arquivos nos roots first-party de produto (`lib/`, `android/`, `assets/`, `ios/`, `linux/`, `macos/`, `tvos/`, `web/`, `windows/`); 2.109 quando `packages/` é incluído. Essas são contagens estruturais reproduzíveis, não uma alegação de que todo arquivo contém UI.
**Git tree auditada:** `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`

---

# 0. Resultado das reauditorias V1 → V2 → V3 → V4 → V5 → V6 → V7 → V8 → V9 → V10

O V1 estava na direção correta, mas não estava completo o suficiente para ser tratado como plano final.

## 0.1 Falha estrutural encontrada no próprio V1

O arquivo PLANO_MESTRE_V1.md termina no começo da Fase 10:

- Fase 10 — Plataformas não-Flutter
- Android
- values/strings.xml
- values-pt-rBR/strings.xml

Não há conclusão da Fase 10, não há fases finais de hardening/release, não há estratégia de rollout/upstream, não há Definition of Done consolidada e não há fechamento de riscos.

O V2 corrigiu o truncamento estrutural do V1. A V3 fechou os blockers de toolchain, native background, ownership de locale e reachability. A V4 adicionou copy remota controlada pelo produto, assets de runtime, notificações de plugin fora do Android, rich text/custom painting e hardcoded directionality. A V5 preserva V1–V4 como histórico e passa a ser a especificação canônica, fechando packaging-generated copy, app-supplied system UI, framework delegates, localization sem BuildContext, harness de testes e verificação do conteúdo localizado dentro dos artefatos publicados.

## 0.2 Baseline upstream reconfirmada

Em 2026-09-25 a release mais recente do Debrify continua sendo v0.10.0-beta.1 e o HEAD verificado continua no commit 9619c10b06ee919cacbe996b30be7739dc09c6d6.

Também foi reconfirmado no código:

- Flutter 3.44.8 no workflow;
- intl ^0.19.0;
- shared_preferences ^2.2.2 no pubspec, resolvendo `shared_preferences 2.5.3` e `shared_preferences_android 2.4.10` no lockfile;
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
3. **O bridge Android por Activity é insuficiente.** Downloads, gravações, alarmes e receivers podem produzir UI/notificações quando o processo Flutter está morto. É necessário um leitor nativo durável da preferência canônica `ui_locale_v1`, apontando para o mesmo backing store usado pelo Dart; não criar segunda chave nem mirror. O Dart/AppLocaleController permanece a autoridade do produto.
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

## 0.6 Novos achados bloqueadores da auditoria V5

A quinta auditoria confrontou o plano V4 com a árvore exata do commit `9619c10b06ee919cacbe996b30be7739dc09c6d6`, o workflow de release e superfícies em que o texto é renderizado pelo sistema operacional ou gerado durante o empacotamento.

1. **A contagem histórica de 1.574 arquivos ficou ambígua.** A árvore Git exata possui 3.199 blobs. Os roots first-party de produto somam 1.736 arquivos; incluindo `packages/`, 2.109. O V5 usa contagens reproduzíveis por Git tree e separa tamanho estrutural de reachability/user-facing coverage.
2. **Linux possui copy user-facing em duas fontes diferentes.** `linux/debrify.desktop` contém `Comment=Debrid client for torrents and streaming`, mas os jobs x64 e arm64 de `.github/workflows/build.yml` geram outro `AppDir/debrify.desktop` inline com `Comment=Modern debrid companion with torrent search and playlist management`. Traduzir apenas o arquivo versionado não traduz o AppImage publicado.
3. **Build scripts/workflows também podem ser fonte de UI.** O scanner precisa seguir copy que nasce em `.github/workflows`, scripts de packaging, CMake/ISS/templates e termina dentro de artifacts. O V5 adiciona `BUILD_GENERATED_PRODUCT_COPY` e provenance no inventário.
4. **Há copy fornecida pelo app em UI renderizada pelo SO.** `MainActivity.kt` cria `RemoteAction` de PiP com `Play`, `Pause` e `Next`; vários `FilePicker.platform.pickFiles/saveFile` fornecem `dialogTitle` como `Save diagnostic logs`, `Choose a Debrify backup` e `Export collection`. Isso não é `SYSTEM_OWNED_UI`: o sistema desenha, mas a copy é responsabilidade do Debrify.
5. **A classificação de ownership precisava separar app-supplied system UI.** O V5 adiciona `APP_SUPPLIED_SYSTEM_UI`; somente chrome realmente escrito pelo SO pode ser excluído como `SYSTEM_OWNED_UI`. Em um file picker, por exemplo, botões do SO podem seguir o idioma do sistema, enquanto o `dialogTitle` fornecido pelo Debrify deve seguir a política do app quando a API permitir.
6. **PiP Android não estava nomeado como surface contract.** `MainActivity.buildPipActions()` usa títulos hardcoded e os entrega também como content description da `RemoteAction`. Esses labels precisam de resources, locale-aware Context e teste durante atualização de PiP já ativo.
7. **A auditoria XML precisa cobrir mais que `text/contentDescription/hint`.** A baseline tem dezenas de layouts TV/player com copy literal. O gate V5 cobre todos os XMLs runtime e tipos/atributos string-bearing, incluindo menus, arrays, plurals, `tooltipText`, `stateDescription`, `accessibilityPaneTitle` e equivalentes futuros, com review de `translatable=false`.
8. **Android resource parity precisa validar placeholders, não só nomes.** `values` e `values-pt-rBR` devem preservar tipo, placeholders `%1$s/%1$d`, ordem/positional form, plurals e flags de formatação. A tradução não pode compilar e ainda quebrar em runtime por mismatch.
9. **A arquitetura proibia BuildContext em services, mas não definia como materializar copy localizada fora da árvore de widgets.** Isso é necessário para `background_downloader`, bootstrap assíncrono e futuros workers. O V5 define `LocalizedCopyResolver`/equivalente: carrega `AppLocalizations` para um `Locale` explícito sem virar segunda autoridade de locale.
10. **Framework localization delegates não estavam explícitos.** O repositório usa `showDatePicker`/`showTimePicker`; todo root produtivo e test harness que renderiza UI precisa dos delegates de app + Material + Widgets + Cupertino. `AppLocalizations.localizationsDelegates` é o caminho preferido quando disponível.
11. **A suite de testes precisa de migração arquitetural.** O upstream possui pelo menos 93 arquivos de teste encontrados com `MaterialApp(` na busca indexada, além de testes que usam `find.text(...)`/`find.byTooltip(...)` em inglês. Sem um `localizedTestApp` comum, a migração pode quebrar testes em massa ou continuar validando apenas inglês.
12. **Acessibilidade nativa possui frases mistas de dado externo + copy local.** Exemplo: `TvSourceBrowserController.kt` monta `"${entry.title}, Playing"`. Isso deve virar formatted resource com placeholder e isolamento bidi quando necessário; `entry.title` continua dado externo.
13. **PWA manifest e DOM têm autoridades diferentes.** `documentElement.lang/dir` pode acompanhar o App language em runtime; `web/manifest.json` e metadata de instalação são estáticos/cacheados pelo browser e não devem ser prometidos como sincronizados instantaneamente. O V5 exige política explícita: metadata neutra ou manifests/HTML locale-aware no servidor.
14. **Windows protocol handler possui strings humanas dentro do Registry.** `URL:Stremio Protocol`, `URL:Magnet Protocol` e `URL:Debrify Protocol` precisam ser classificadas separadamente dos identifiers técnicos `stremio/magnet/debrify` e `URL Protocol`. Nunca traduzir IDs/keys; localizar a descrição humana se ela fizer parte da UI do shell ou allowlistar com evidência.
15. **Completude precisa ser verificada dentro do artifact final.** Source parity não prova que APK/IPA/DMG/EXE/AppImage carregou a localização. O V5 adiciona Gate N para inspecionar artifacts publicados: Android resources, Apple `.lproj`, Windows languages/custom messages, Linux `.desktop` localizado e Web shell quando buildado.

Esses achados não invalidam a arquitetura V4; eles fecham rotas de escape que só aparecem quando se segue a copy até o artifact ou até uma superfície do sistema operacional.

---
## 0.7 Novos achados bloqueadores da auditoria V6

A sexta auditoria partiu da V5 e procurou classes de falha que ainda poderiam produzir experiência parcialmente inglesa ou inconsistente mesmo com ARB/resources/artifacts em verde.

1. **Nomes de idiomas são copy própria em vários caminhos e ainda existem como mapas ingleses duplicados.** `lib/models/stremio_subtitle.dart` e `android/.../StremioSubtitleService.kt` possuem tabelas próprias `eng -> English`, `spa -> Spanish`, `por -> Portuguese`; Settings/Filters também mantêm labels ingleses. Código ISO/BCP-47 é identidade/dado; o nome apresentado é copy localizada. A V6 cria contrato único de resolução sem usar o label como identidade.
2. **Formatação de saída locale-aware não resolve entrada humana.** `collection_editor_screen.dart` valida rating com `double.tryParse`, fazendo `8,5` falhar em pt-BR. A V6 separa parsing de entrada humana locale-sensitive de tokens técnicos/protocolos invariantes.
3. **O teclado próprio de TV é uma superfície i18n de primeira classe.** `TvTextField` gera `Search/Go/Next/Send/Done`, vários callers passam `keyboardSubmitLabel` em inglês e `tv_keyboard.dart` contém ações como `Clear`. Esses sinks entram explicitamente no inventário/gates.
4. **Mensagens humanas não podem ser o contrato entre runtimes.** Platform channels e services possuem `result.error(..., "English message", ...)`, `e.message` e exceptions inglesas. A V6 exige reason/error codes estáveis + argumentos; a camada que apresenta ao usuário resolve copy pelo locale efetivo.
5. **O Remote cruza dispositivos com locale device-local, mas partes do protocolo carregam `message`/labels humanos.** Telefone em pt-BR e TV em inglês — ou o inverso — não podem herdar a língua do emissor. Resultados cross-device passam a transportar `resultCode/reasonCode` + dados estruturados; cada dispositivo localiza no destino.
6. **Artifact inspection não substitui runtime real para superfícies OS/hardware-owned.** PiP, notification channels, FilePicker, permission prompts, Top Shelf, TV input/focus e screen readers precisam de smoke de runtime em target representativo. A V6 adiciona Gate O.
7. **Catálogo de tradução também precisa de higiene de ciclo de vida.** Além de key parity, CI deve detectar keys ARB órfãs/não alcançadas e exceções obsoletas para que a migração não acumule copy morta ou traduções que parecem cobertura sem qualquer sink real.


## 0.8 Novos achados bloqueadores da auditoria V7

A V7 revalida a mesma baseline exata da V6 e acrescenta classes de falha que não são capturadas por um scanner focado apenas em literais traduzíveis:

1. **Composição manual de listas user-facing.** Há caminhos reais que usam `.join(', ')`, `.join(' · ')` e `.join('\n')` para formar apresentação. Nem todo `join` precisa ser localizado, mas todo `join` que alcança UI precisa ser classificado como lista natural localizada, metadata visual estruturada, bloco multiline ou dado técnico.
2. **Truncamento e transformação por code unit.** A baseline contém `String.length` + `substring` em nomes exibidos e capitalização por `s[0]`. Isso pode cortar surrogate pairs, ZWJ emoji e combining marks. UI deve operar por grapheme cluster quando a intenção é “caractere percebido pelo usuário”.
3. **Casing localizado/grapheme-safe.** `toUpperCase/toLowerCase` continuam permitidos para tokens técnicos; para copy/dados visíveis devem ser evitados ou aplicados por uma política explícita que não quebre graphemes nem assuma regras inglesas.
4. **Sinks de texto fora da árvore de widgets.** Clipboard, conteúdo gerado para share/export/report e nomes/títulos sugeridos por dialogs/plugins podem ser user-facing mesmo sem passar por `Text(...)`. O inventário passa a classificá-los explicitamente.
5. **Coerência documental.** Arquitetura, quality gates e matriz de testes deixam de carregar cabeçalhos históricos “V4 + apêndices” e passam a identificar a versão canônica V7, reduzindo ambiguidade para implementação automatizada.

Evidências detalhadas ficam em `docs/AUDITORIA_V7.md`.

## 0.9 Novos achados bloqueadores da auditoria V8

A oitava auditoria revalidou a árvore Git inteira do baseline (3.199 blobs; resposta recursive tree não truncada), os documentos normativos do projeto de planejamento e os contratos reais Dart↔Android do upstream. A baseline continua exatamente em `v0.10.0-beta.1` / `9619c10b06ee919cacbe996b30be7739dc09c6d6`.

1. **Gate P estava fora do plano canônico.** A Definition of Done da V7 exigia `Gate P — Unicode/composition/outbound sinks`, e `docs/AUDITORIA_V7.md`/`docs/CI_QUALITY_GATES.md` o definiam, mas a seção 11 de `PLANO_MESTRE_V7.md` terminava no Gate O. Um agente que executasse somente o plano canônico poderia promover PT-BR sem executar P. A V8 incorpora P no contrato normativo.
2. **Os IDs dos gates divergiam entre documentos.** O plano usava `Gate A/B/.../O`, enquanto `docs/CI_QUALITY_GATES.md` mantinha `Gate 1..14` e aliases parciais como `Gate 12 / Gate L`. A V8 define um único registro canônico: `0, A..Q`; nenhum documento normativo pode renumerar ou criar alias.
3. **Faltava um gate executável de drift do upstream.** “Revalidar/rebasear” aparecia como instrução, mas não existia um blocker que comparasse o SHA auditado com a nova base antes de cada rebase/release. A V8 adiciona Gate Q e um manifest machine-readable do baseline.
4. **O contrato “aceitar BCP-47” era mais amplo que o tipo `Locale` preserva.** Para locale de UI, Flutter representa language/script/region, mas não mantém variantes, extensions ou private-use como identidade completa. A V8 define `ProductLocaleId = language[-Script][-REGION]` e proíbe perda silenciosa de subtags.
5. **O backing store Android podia ser mais preciso.** Na baseline, `DevicePreferences` usa o `SharedPreferences.getInstance()` legado; o código nativo já lê `FlutterSharedPreferences` com prefixo físico `flutter.`. Para este baseline, `ui_locale_v1` corresponde exatamente a `FlutterSharedPreferences / flutter.ui_locale_v1`. O adapter continua isolando esse detalhe para futura troca de backend.
6. **Links oficiais externos e QR tinham uma promessa de locale ambígua.** Um browser externo pode negociar pela língua do browser/SO, não pelo override interno do Debrify; um QR aberto em outro dispositivo também não herda o App language. A V8 exige locale explícito na URL quando suportado, ou landing neutra com seletor/fallback documentado.
7. **Caches de apresentação precisavam de uma regra geral.** Settings Search, remote copy e `LocalizedCopyResolver` já tinham cuidados locais, mas a política não proibia globalmente cachear texto já localizado sem locale/epoch na chave. A V8 torna isso invariant e adiciona testes de troca de idioma sem restart.
8. **A língua da árvore de acessibilidade precisava de prova própria.** No Flutter 3.44.8, `Localizations` em nível de aplicação não atribui `localeForSubtree` ao `Semantics` gerado; esse atributo é aplicado no caminho não-application-level. Como App language pode divergir do system locale, a V8 exige inspeção da Semantics tree e, se necessário, um wrapper compartilhado com `Semantics(localeForSubtree: effectiveLocale)`, além de smoke real de screen reader.

Evidências detalhadas ficam em `docs/AUDITORIA_V8.md`.

## 0.10 Novos achados bloqueadores da auditoria V9

A nona auditoria revalidou a mesma baseline exata da V8 e confirmou que o `main` do upstream continua idêntico a `9619c10b06ee919cacbe996b30be7739dc09c6d6`. O Gate Q, portanto, permanece verde para a árvore auditada. A V9 não amplia artificialmente o escopo: ela torna executáveis classes de risco que ainda estavam cobertas apenas de forma genérica.

1. **Calendário/data ainda possui presentation copy inglesa codificada manualmente.** A baseline contém `Monday..Sunday`, `January..December`, `Today/Tomorrow/Yesterday` e abreviações inglesas em caminhos reais como `trakt_calendar_day_sheet.dart`, `trakt_calendar_screen.dart`, `iptv_epg_panel.dart`, `recordings_page.dart`, `spotlight_live_timeline.dart` e `trakt_episode_model.dart`. “Formatters são locale-aware” não basta: o inventário precisa encontrar tabelas manuais de calendário e montagem de data.
2. **Há formatação de apresentação dentro de models/services.** Exemplos confirmados incluem `RdUser.formattedExpiration` e helpers `_formatDate`/`toStringAsFixed` em models de contas. Isso viola a fronteira semântica se o model devolver a string final ao usuário. A V9 exige que domínio exponha `DateTime`/número/estado e presentation faça a formatação.
3. **Tempo civil, timecode, timestamp técnico e calendário de provider são semânticas diferentes.** A baseline mistura relógios que respeitam `MediaQuery.alwaysUse24HourFormat`, relógios forçados a 24h e timecodes `hh:mm:ss`. Também há chunking de Trakt deliberadamente alinhado à segunda-feira. A V9 proíbe “localizar tudo”: civil time/data seguem locale/preferência; media timecode, API/path/filename/diagnóstico e regras do provider permanecem invariantes quando essa é a semântica.
4. **Troca de locale possui risco de race assíncrona.** A V8 cobre cache stale, mas não impedia uma requisição/resolver iniciado em inglês de terminar depois da troca para pt-BR e materializar copy velha. A V9 introduz `localeEpoch`/generation e regra de commit condicional para trabalho assíncrono locale-sensitive.
5. **Android per-app language estava como decisão aberta.** A baseline não possui `android:localeConfig`, `LocaleManager`, `AppCompatDelegate.setApplicationLocales` nem `generateLocaleConfig`. Para o primeiro release pt-BR, a V9 fecha a decisão: **não criar uma segunda autoridade do SO**. `AppLocaleController/ui_locale_v1` permanece a única autoridade do produto. Integração futura com Android per-app language exige migração arquitetural dedicada.
6. **Fallback de idioma não pode mudar silenciosamente ao adicionar locales.** `pt -> pt-BR` deve ser uma política explícita/versionada, não um efeito colateral de “há apenas um português shipping”. Adicionar `pt-PT` no futuro não pode mudar automaticamente a resolução de instalações existentes.
7. **Busca accent-friendly precisa de conformance Unicode composta/decomposta.** A V8 centraliza a normalização, mas a V9 exige corpus NFC/NFD/combining marks e impede implementação por mapas ad hoc que funcione apenas para os exemplos mais óbvios.
8. **Collation não pode ser simulada por `String.compareTo`/lowercase/search-fold.** Se o produto realmente exigir ordenação alfabética locale-aware, deve existir um collator adequado por plataforma/biblioteca com teste. Caso contrário, manter ordem de produto/provider estável é mais correto que declarar falsa collation.
9. **Language attribution de acessibilidade precisa respeitar mixed-language ownership.** Um `Semantics(localeForSubtree: appLocale)` aplicado cegamente ao app inteiro pode rotular filename, URL, título externo ou user data como pt-BR. A V9 torna o wrapper scoped por ownership.
10. **App language é mutação de apresentação, não reset funcional.** Trocar en ↔ pt-BR não pode perder navigation stack, playback/source/position, D-pad focus sem necessidade, formulário não salvo, download/recording ou pairing remoto. Recriação nativa inevitável precisa restaurar esse estado.
11. **Packages runtime vendorizados precisam de regra de scan operacional.** A árvore auditada possui `packages/` com Dart e código nativo Android/iOS/macOS/Linux/Windows. Gates D/H/N/Q passam a percorrer roots runtime de packages e exigem classificação `FIRST_PARTY_FORK`, `VENDORED_THIRD_PARTY` ou `GENERATED`.

12. **Glossário canônico não pode carregar decisão editorial aberta.** A baseline do planejamento ainda dizia que “Player” poderia virar “Reprodutor”. A V9 fecha a copy PT-BR em **Reprodutor** e mantém `player` apenas como token técnico/código quando aplicável.

Evidências detalhadas e hotspots confirmados ficam em `docs/AUDITORIA_V9.md` e `docs/AUDIT_HOTSPOTS_V9.json`.


## 0.11 Novos achados bloqueadores da auditoria V10

A décima auditoria manteve a mesma baseline exata — o `main` upstream continua idêntico a `9619c10b06ee919cacbe996b30be7739dc09c6d6` — e procurou uma classe diferente de fuga: **copy de apresentação que não nasce no sink visual**, mas fica armazenada em enums, extensions, models e services e só depois é consumida pela UI.

1. **Registries/enums de apresentação eram um blind spot de dataflow.** Há app-owned copy em propriedades como `label`, `description`, `displayName`, `subtitle` e getters equivalentes, consumidas mais tarde por Settings/pickers. Casos confirmados: `ContentDisplayMatchMode`, `AndroidVideoRendererMode`, external-player models, `MetadataCategory`, `PlaylistViewMode` e o catálogo de launch idents. Um scanner focado no literal dentro de `Text(...)` pode não ligar a origem ao sink.
2. **`CalendarTimeFormat` ainda mistura identidade e apresentação.** `lib/widgets/calendar_display_preferences.dart` guarda `Device default`, `12-hour`, `24-hour` dentro do enum e implementa o modo 12h com `AM/PM` manual. O enum precisa manter somente identidade/semântica; label e hora civil pertencem ao mapper/formatter localizado.
3. **`StremioTvNowPlaying` materializa frase e relógio dentro do model.** `progressText` retorna `Ended` / `Ends at ...` e `formatTime` força 12h + `AM/PM`. Isso viola diretamente a fronteira V9 “domain retorna dados, presentation retorna texto”.
4. **Debrify TV ainda possui relógio civil manual fora dos hotspots V9.** `lib/screens/debrify_tv/widgets/stats_tile.dart` monta `HH:mm` por `padLeft` e combina com `Search snapshot` / `Queue prepared...`. Esse caminho entra explicitamente no inventário temporal.
5. **O picker de hora da TV usa bem o Material para AM/PM, mas seu chrome continua inglês.** `lib/widgets/tv_time_picker.dart` ainda possui `Pick a time`, instrução de D-pad, `Cancel` e `Set`. O arquivo passa a ser hotspot explícito para não depender apenas de detecção genérica.
6. **Compact-number presentation também escapa por models/services.** `YoutubeVideo.formattedViews`, `RedditVideoPost.formattedScore` e `LemmyVideoPost.formattedScore` fazem `toStringAsFixed`/sufixos na camada de dados; `formattedViews` inclui inclusive a palavra inglesa `views`. Contagem/compact notation e gramática devem ser resolvidas na apresentação com locale explícito.
7. **A prova de “100% de cobertura” precisava de fechamento por conjuntos.** A V9 exige inventário determinístico, mas não formaliza que todo blob runtime da tree auditada esteja exatamente em uma classe. A V10 exige `RuntimeSurfaceUniverse == ClassifiedReachable ∪ ExcludedWithEvidence`, com conjuntos disjuntos e zero path sem decisão.
8. **Gate D ganha análise de origem → sink, não apenas pattern matching.** Campos/getters/records/enums que carregam nomes de apresentação devem ser classificados quando alcançam UI, mesmo que o literal esteja vários frames/calls antes do widget.
9. **Gate F ganha detector temporal/compact mais semântico.** Padrões `hour % 12`, `hour >= 12`, literais `AM/PM`, `padLeft` em relógio civil e compactação manual com `toStringAsFixed + K/M/B` entram como findings quando alcançam UI.
10. **Nenhum gate novo é criado.** Os achados V10 endurecem D, E, F, P e Q e preservam o registro canônico `0, A..Q`.

Evidências detalhadas ficam em `docs/AUDITORIA_V10.md` e `docs/AUDIT_HOTSPOTS_V10.json`.

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

Exemplos de `SYSTEM_OWNED_UI`: botões/chrome do file picker, share sheet e permission prompt quando o texto é fornecido pelo SO.

Adicionar `APP_SUPPLIED_SYSTEM_UI` para copy do Debrify que é desenhada pelo SO/plugin, e `BUILD_GENERATED_PRODUCT_COPY` para copy criada por workflow/script e empacotada no artifact.

Exemplos de `APP_SUPPLIED_SYSTEM_UI`: `dialogTitle` passado ao FilePicker e títulos/content descriptions de Android `RemoteAction` PiP.

Regra: ownership é determinado por **quem fornece o texto**, não por quem desenha a janela. O app localiza toda copy que ele controla. Terceiros não entram em “100% de copy própria” sem prova de ownership, mas qualquer string que o Debrify injete neles continua em escopo.

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
| `APP_SUPPLIED_SYSTEM_UI` | PiP RemoteAction, FilePicker dialogTitle | localizar a parte fornecida pelo app; documentar o chrome OS-owned |
| `BUILD_GENERATED_PRODUCT_COPY` | `.desktop`/metadata produzidos no workflow | fonte canônica locale-aware + teste no artifact final |

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

Para o fluxo WebDAV, como o app oferece `Setup guide` e um QR fixo para `debrify.tv/guides/webdav-sync/`, o destino deve ter política de locale.

Regra V8 para qualquer `OFFICIAL_PRODUCT_CONTENT` aberto fora do app:

- se a URL oficial suporta locale explícito, o link disparado pelo app deve carregar o `ProductLocaleId` efetivo por path/query/contrato equivalente;
- não assumir que `Accept-Language` do browser externo representa o override interno do Debrify;
- QR aberto em outro dispositivo não herda o App language do dispositivo de origem;
- QR fixo deve apontar para landing neutra que preserve a tarefa e ofereça seleção/fallback de idioma, ou para um redirect que determine locale sem prometer sincronização com o app de origem;
- o inventário registra owner, destino, mecanismo de locale e fallback para todo domínio oficial acionado pelo produto.

Release notes podem ser publicadas em formato multilíngue ou via source locale-aware. Se o produto decidir que release notes são editoriais e não bloqueiam o shipping locale, essa exceção precisa ser explícita e mensurada — nunca implícita.


---

## 2.12 Código de idioma é identidade; nome de idioma é apresentação

Para áudio, legendas, filtros e qualquer outra UI:

- ISO 639/BCP-47/canonical language id permanece estável para matching, persistência e protocolo;
- `English`, `Portuguese (Brazil)`, `Spanish` etc. são copy de apresentação quando gerados pelo Debrify;
- label vindo explicitamente de addon/arquivo/provider continua external data, salvo quando o produto gera fallback próprio;
- nenhum branch compara o nome localizado do idioma;
- nomes de fallback próprios devem ser resolvidos por uma fonte semântica comum, não por mapas ingleses duplicados em Dart/Kotlin.

A implementação deve oferecer um catálogo/resolver testável de display names com paridade entre Flutter e Android nativo.

## 2.13 Copy localizada não atravessa fronteira de protocolo como autoridade

Entre Dart↔native e, principalmente, telefone↔TV:

- transportar `reasonCode`/`resultCode` estável;
- transportar argumentos estruturados necessários à mensagem;
- localizar somente no runtime/dispositivo que efetivamente renderiza a UI;
- strings externas/user data podem atravessar quando são dados, nunca como comando/estado;
- durante migração, payload legado com `message` pode ser aceito como fallback de compatibilidade, mas não se torna a API canônica.

Isso preserva o princípio de App language device-local e evita mixed-language causado pelo emissor.

## 2.14 Entrada humana localizada é diferente de token técnico

O locale afeta parsing de campos destinados a números humanos, como rating/percentual configurável. Ele **não** altera:

- IP;
- URL;
- porta;
- PIN/código;
- IDs;
- hashes;
- timestamps/formato exigido por schema;
- valores de protocolo/API.

A borda de apresentação converte entrada humana para valor canônico tipado antes de persistir.

## 2.15 Troca de App language é uma mutação de apresentação

Alterar App language deve mudar somente apresentação/semântica de linguagem e caches derivados.

Não deve, por efeito colateral:

- trocar metadata/audio/subtitle/region;
- resetar navigation stack;
- reiniciar player ou trocar source;
- zerar posição de playback;
- cancelar download/recording;
- limpar pairing/remote session;
- descartar input/formulário não salvo;
- deslocar foco TV para um destino arbitrário.

Se uma plataforma exigir recreation para aplicar recursos nativos, a implementação preserva/restaura o estado funcional antes/depois da recreation e registra a exceção no Gate O.

## 2.16 Fallback de locale é política versionada, não heurística do conjunto shipping

Criar uma política explícita equivalente a:

    LocaleFallbackPolicy
      languageDefaults:
        pt -> pt-BR
        en -> en

Regras:

- `ShippingLocales` diz **o que pode ser selecionado**;
- `LocaleFallbackPolicy` diz **como uma preferência do SO é resolvida**;
- adicionar/remover locale shipping não altera automaticamente `languageDefaults`;
- `pt-PT` nunca cai em pt-BR somente por compartilhar `languageCode`;
- listas do SO são avaliadas em ordem, preservando preferências subsequentes quando a primeira não possui match permitido;
- qualquer mudança de fallback policy é decisão de produto versionada, com teste de upgrade.

## 2.17 Presentation state assíncrono é locale-versioned

Qualquer trabalho assíncrono que materialize copy, índice localizado, menu, notification payload ou presentation model captura:

    ProductLocaleId
    localeEpoch

A resposta pode atualizar dados semânticos/cache neutro; porém só pode publicar **copy materializada** se `localeEpoch` ainda for o atual.

Isso vale para:

- remote product copy/config;
- `LocalizedCopyResolver`;
- Settings Search/index assíncrono;
- menus/overlays long-lived;
- plugin/background copy configurável em runtime;
- qualquer future/stream que transforme semantic state em texto.

Troca de locale incrementa o epoch mesmo quando a navegação/tela permanece viva.


## 2.18 Copy declarativa continua sendo apresentação

Um literal app-owned não deixa de ser copy apenas porque está armazenado em:

- enum constructor;
- extension getter;
- model/service getter;
- record/tuple;
- registry/config object;
- `static const` option table;
- field `label/title/subtitle/description/helpText/emptyText/statusText/displayName`.

Se o valor alcança UI, accessibility, notification, chooser, clipboard/share/report ou outra superfície humana, ele recebe a mesma classificação de ownership/reachability de um literal diretamente em `Text(...)`.

Regra arquitetural:

- enum/model mantém identidade estável e dados canônicos;
- presentation mapper recebe a identidade + `AppLocalizations`/locale explícito;
- brand/external/user data pode continuar como dado, mas precisa de classificação;
- product copy não fica congelada em getters de domínio para “facilitar” a UI.

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
- locale de arquivo gerado usa convenção do gen_l10n (`pt_BR`); persistência do produto usa `ProductLocaleId` BCP-47-base (`pt-BR`).

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
- aceitar somente `ProductLocaleId` canônico (`language[-Script][-REGION]`) para override manual;
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

Para a identidade de **locale de UI do produto**, usar um contrato deliberadamente menor que “qualquer BCP-47”:

    ProductLocaleId := language[-Script][-REGION]
    persisted       := system | ProductLocaleId

O contrato cobre sem perda os subtags representáveis por `Locale.fromSubtags(languageCode, scriptCode, countryCode)`. Não persistir silenciosamente variantes, extensions ou private-use como se Flutter pudesse round-trip completo, por exemplo `ca-ES-valencia`, `en-US-u-hc-h12` ou `x-private`.

Regras:

- aceitar/canonicalizar somente o subconjunto suportado para `ShippingLocales`;
- aliases legados conhecidos podem migrar de forma explícita e testada;
- locale do SO que contenha informação além do modelo do produto pode ser usado para matching pelos subtags-base suportados, sem regravar uma tag truncada como preferência manual;
- preferências ortogonais codificáveis em extensions, como hour-cycle, continuam sob autoridade própria da plataforma e não viram parte de `ui_locale_v1`;
- valor persistido com subtags não suportados é rejeitado/faz fallback seguro, nunca normalizado por perda silenciosa;
- manter conversores de representação separados: ARB/Dart `pt_BR`, BCP-style product id `pt-BR`, Android resources `pt-rBR`, Linux `pt_BR`, Apple `pt-BR`.

Construir `Locale` com `languageCode/scriptCode/countryCode` somente depois da validação/canonicalização de `ProductLocaleId`.

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
- `ProductLocaleId` e subtags-base vindos do SO são canonicalizados antes do match;
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
- listener do AppLocaleController;
- `AppLocalizations.localizationsDelegates` (ou delegates equivalentes explícitos) cobrindo app + Material + Widgets + Cupertino;
- locale resolution comum, sem roots de bootstrap com delegates incompletos.

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



## 3.12 LocalizedCopyResolver para código sem BuildContext

O invariant continua válido: services/models não recebem `BuildContext` apenas para traduzir texto. Porém, código assíncrono fora da árvore precisa de uma forma única de materializar copy.

Criar um adaptador pequeno, por exemplo `LocalizedCopyResolver`, com estas regras:

- recebe um `Locale` efetivo explícito vindo do `AppLocaleController`/resolver; não escolhe locale sozinho;
- carrega/obtém `AppLocalizations` pelo delegate gerado (`AppLocalizations.delegate.load(locale)` ou API equivalente da versão pinada);
- pode expor um objeto estreito de mensagens para workers/plugins quando injetar `AppLocalizations` inteiro for excessivo;
- não persiste locale e não observa plataforma por conta própria;
- não usa `Intl.defaultLocale` como autoridade paralela;
- cache, se existir, é keyed pelo `ProductLocaleId` efetivo e invalidado/trocado quando o locale muda;
- copy configurada em plugin/background é reconfigurada após mudança de App language quando a plataforma permite;
- callbacks que executam em isolate/processo sem acesso à instância viva usam locale persistido canônico + contrato explícito da plataforma, nunca um mirror ad hoc.

Casos mínimos:

- `background_downloader` non-Android;
- startup/recovery helpers que montem copy antes de um widget;
- futuras tarefas/headless callbacks com user-facing notifications;
- testes system en + override pt-BR e system pt-BR + override en.

Esse resolver é um **materializador de copy**, não uma segunda autoridade de locale.

---
## 3.13 Resolvers semânticos

Adicionar componentes conceituais equivalentes a:

    LanguageDisplayNameResolver
    LocalizedInputParser
    UiFailureResolver

Regras:

- recebem locale efetivo explicitamente;
- não persistem locale;
- não usam `Intl.defaultLocale` como autoridade global;
- retornam copy apenas na borda de apresentação;
- preservam IDs/codes tipados por baixo;
- podem ser exercitados sem `BuildContext`;
- possuem testes en/pt-BR e cross-runtime quando houver implementação nativa equivalente.

`LanguageDisplayNameResolver` resolve nomes gerados pelo produto a partir de código canônico. `LocalizedInputParser` converte entrada humana locale-sensitive para número tipado. `UiFailureResolver` mapeia reason codes estáveis para AppLocalizations.

## 3.14 Cache de apresentação e troca de locale em runtime

Texto já materializado por `AppLocalizations`, formatter ou resolver é **presentation state**, não dado canônico.

Regras:

- preferir cachear estado semântico + argumentos, não a frase localizada;
- qualquer cache inevitável de apresentação deve ser keyed por `ProductLocaleId` efetivo (e, quando necessário, direção/format preference) e invalidado atomicamente na troca de locale;
- singleton/static cache de `String` localizada sem locale na chave é finding;
- overlays, dialogs, menus, search indexes e views nativas mantidas vivas devem atualizar ou ser recriadas de maneira segura após mudança de App language;
- background/headless lê o locale persistido no momento da apresentação, não uma cópia capturada indefinidamente no startup;
- testes devem alternar en → pt-BR → en **sem restart** em superfícies long-lived e provar ausência de stale copy.

Gate E inventaria acoplamento/persistência indevida; Gate L cobre cache remoto; Gate P cobre composição/outbound; o scanner também deve reportar caches de apresentação não locale-keyed.

## 3.15 LocaleEpoch e commit condicional

`AppLocaleController` expõe uma geração monotônica de apresentação, conceitualmente:

    int get localeEpoch;
    LocaleSnapshot get snapshot; // effective ProductLocaleId + epoch

Ao mudar o locale efetivo:

1. persistir/validar a intenção;
2. atualizar effective locale;
3. incrementar epoch;
4. notificar Flutter/native adapters;
5. invalidar somente caches de apresentação.

Consumer assíncrono:

    final snapshot = appLocale.snapshot;
    final semanticPayload = await repository.load();
    if (!appLocale.isCurrent(snapshot)) return;
    publish(localize(semanticPayload, snapshot.locale));

Não descartar semantic payload útil apenas porque o locale mudou; impedir apenas o commit de presentation materialized sob geração velha.

Testes obrigatórios usam completions fora de ordem:

    en request starts
    switch -> pt-BR
    pt-BR request completes
    en request completes late
    UI must remain pt-BR

e o inverso no flip pt-BR -> en.

## 3.16 LocaleFallbackPolicy separado do catálogo

O resolver de locale deve receber duas estruturas distintas:

- catálogo de locales shipping;
- fallback policy versionada.

O catálogo não pode inferir sozinho defaults de idioma. Testes de upgrade precisam provar que a inclusão futura de `pt-PT` não muda, sem decisão explícita, o significado persistido de `system`/preferências já existentes.


## 3.17 Localized display adapters para options/enums

Para options persistidas, separar três conceitos:

    stable id/storage key
    semantic enum/value
    localized presentation

Exemplo-alvo:

    ContentDisplayMatchMode.frameRate.storageKey  // frame_rate
    l10n.contentDisplayMatchFrameRate             // "Match frame rate" / PT-BR

Nunca localizar `storageKey`, enum `name`, executable, bundle id, protocol token ou command argument.

Settings, Settings Search, pickers, accessibility e qualquer summary devem consumir o mesmo mapper localizado; não manter uma tabela inglesa para a tela e outra para busca.

Para registries grandes (launch idents/themes etc.), permitir que brand names estáveis permaneçam brand, mas descrição/pitch app-owned é localizada quando exibida.

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

Conformance V9 obrigatória:

- a implementação central escolhida deve tratar de forma determinística texto Unicode equivalente em NFC/NFD;
- `Configuração` composta e sua forma decomposta com combining marks produzem o mesmo search key para PT-BR;
- repetir o teste para `áudio`, `vídeo`, `conexão` e `reprodução`;
- não espalhar tabelas de substituição por widget;
- se uma dependência/tabela especializada for adotada, pin/versionar e cobrir por corpus; não assumir comportamento não testado;
- normalização de busca nunca modifica o valor exibido/persistido;
- search-fold não é collator e não pode ser reutilizado como implementação de ordenação locale-aware.

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

## 4.12 Language display names — remover mapas ingleses duplicados

Hotspots confirmados:

- `lib/models/stremio_subtitle.dart::_formatLanguageCode`;
- `android/app/src/main/kotlin/com/debrify/app/subtitle/StremioSubtitleService.kt::languageNames`;
- `lib/screens/settings/filter_settings_page.dart::_languageOptions`;
- aliases/labels de áudio e legenda em `settings_screen.dart`.

Refatorar para:

    canonical language code -> semantic language key -> localized display name

Aceite:

- en mostra `English`, `Portuguese (Brazil)`, etc.;
- pt-BR mostra equivalentes PT-BR;
- matching continua usando code, não display name;
- `pt-BR`/`por-br` aliases convergem para a mesma identidade quando semanticamente equivalentes;
- unknown code cai para representação técnica segura do código, não para uma palavra inglesa inventada;
- Dart e Android nativo têm tabela de identidade/paridade testada.

## 4.13 Erros cross-runtime e raw exception leakage

Todo boundary deve distinguir:

    machine reason/code
    structured args
    diagnostic detail
    localized presentation

Exemplos que entram na migração:

- Android `result.error("busy", "A file picker is already open.", ...)`;
- `picker_unavailable`;
- `e.message` atravessando MethodChannel;
- `PlatformException.message`;
- Dart `Exception('...')` cuja string chega a SnackBar/Dialog;
- mensagem de provider remoto que hoje pode chegar crua à UI.

Regra: a UI nunca depende do texto do exception para decidir comportamento. Mensagem externa pode ser exibida como detalhe técnico/terceiro quando útil, sempre sob contexto localizado e com redaction apropriada.

## 4.14 Remote control — resultado semântico por dispositivo

O Remote precisa ser auditado como protocolo multi-locale.

Migrar resultados como:

    { ok: false, resultCode: "admin_profile_required", args: {...} }

em vez de tratar:

    { ok: false, message: "Open an Admin profile on the TV, then resend" }

como contrato final.

Cobrir:

- `remote_chunked_send.dart`;
- outcomes em `remote_command_router.dart`;
- `remote_control_screen.dart` e demais SnackBars/Dialogs que exibem `message`;
- config/addon/channel transfer;
- pairing/recovery;
- compatibilidade com peers na versão anterior.

Teste obrigatório: emissor en + receptor pt-BR e emissor pt-BR + receptor en produzem UI na língua **do receptor**.

## 4.15 TV keyboard/action copy

Inventariar e localizar explicitamente:

- `keyboardSubmitLabel`;
- defaults `Search`, `Go`, `Next`, `Send`, `Done` em `TvTextField`;
- ações do `tv_keyboard.dart`, incluindo `Clear` e semantics de paste/voice/backspace;
- `suggestionsLabel`;
- hints/instruções dentro do overlay de teclado.

Labels técnicos/iconográficos como `?123` e `ABC` devem ser classificados conscientemente; não entram em allowlist por acidente.

## 4.16 Hotspots temporais e de calendário confirmados

A Fase 0 deve registrar explicitamente estes paths do baseline, além do scan completo:

- `lib/widgets/trakt_calendar_day_sheet.dart` — weekdays/months ingleses;
- `lib/screens/trakt_calendar_screen.dart` — month/headline/short-day presentation;
- `lib/widgets/iptv/iptv_epg_panel.dart` — Today/Tomorrow/Yesterday + weekdays/months;
- `lib/screens/settings/recordings_page.dart` — weekdays e data manual;
- `lib/widgets/iptv/spotlight/spotlight_live_timeline.dart` — timeline date labels;
- `lib/services/trakt/trakt_episode_model.dart` — month abbreviation de apresentação;
- `lib/utils/formatters.dart` — `DateFormat('MMM dd, yyyy')` sem locale explícito;
- `lib/models/rd_user.dart` + `account_status_widget.dart` — `formattedExpiration` vindo do model;
- models de AllDebrid/Premiumize/TorBox e telas WebDAV/Stremio Addons — datas/números formatados manualmente;
- `iptv_stage_panel.dart` e `iptv_console_widgets.dart` — `alwaysUse24HourFormat: true` a classificar.

O fato de um path estar nesta lista não significa “traduzir tudo”. Cada ocorrência é classificada pela taxonomia temporal da seção 6.13.


## 4.17 Declarative presentation registries — hotspots V10

Tratar como uma unidade de migração os app-owned labels/descriptions que hoje vivem fora do sink visual.

Hotspots confirmados:

- `lib/models/content_display_match_mode.dart`;
- `lib/models/android_video_renderer_mode.dart`;
- `lib/models/external_player.dart`;
- `lib/models/windows_external_player.dart`;
- `lib/models/linux_external_player.dart`;
- `lib/models/ios_external_player.dart`;
- `lib/models/metadata_preferences.dart`;
- `lib/models/playlist_view_mode.dart`;
- `lib/widgets/launch/**`.

Regras:

- preservar `storageKey`/enum value/command/executable;
- migrar somente PRODUCT_COPY;
- nomes de produto/brand como VLC, IINA, mpv, MPC-HC, PotPlayer etc. permanecem brand;
- labels genéricos como System Default, Custom App, Custom Command e descriptions são localizados;
- Settings Search não indexa o antigo label inglês por acidente; inglês extra só permanece quando classificado como `LEGACY_EN_ALIAS`;
- testes provam round-trip de storage sem depender da tradução.

## 4.18 Temporal/presentation hotspots adicionais V10

### CalendarTimeFormat

`lib/widgets/calendar_display_preferences.dart`:

- remover `label` inglês do enum;
- localizar `Device default`, `12-hour` e `24-hour` por mapper;
- para `device`, continuar respeitando `MediaQuery.alwaysUse24HourFormat`;
- para opções explícitas 12h/24h, usar formatter/framework que produza a convenção correta sem concatenar `AM/PM` manualmente;
- persistência continua por `CalendarTimeFormat.name`/id estável, nunca pelo label.

### StremioTvNowPlaying

`lib/models/stremio_tv/stremio_tv_now_playing.dart`:

- remover `progressText` como presentation API do model;
- expor somente `slotEnd`, estado ended/active e demais dados canônicos;
- UI resolve `Ended` / `Ends at {time}` via ARB + formatter civil-time;
- não forçar 12h por implementação local.

### Debrify TV stats

`lib/screens/debrify_tv/widgets/stats_tile.dart`:

- `Search snapshot`, `Queue prepared`, `Last search` via ARB;
- contagem via formatter/plural quando a gramática exigir;
- `lastSearchedAt` via formatter de `CIVIL_TIME`, não `padLeft` manual.

### TV time picker

`lib/widgets/tv_time_picker.dart`:

- manter uso de `MaterialLocalizations.anteMeridiemAbbreviation/postMeridiemAbbreviation`;
- localizar fallback `Pick a time`, instrução D-pad e ações;
- preferir labels Material existentes quando semanticamente equivalentes; usar AppLocalizations para copy específica do Debrify;
- testar en/pt-BR, 12h/24h, D-pad e text scale.

## 4.19 Compact metrics fora da presentation layer

Hotspots confirmados:

- `lib/services/youtube_service.dart::formattedViews`;
- `lib/services/reddit_service.dart::formattedScore`;
- `lib/services/lemmy_service.dart::formattedScore`.

Regras:

- service/model expõe número canônico;
- presentation usa `NumberFormat`/formatter comum com locale explícito;
- palavras como `views` são ICU/ARB, não sufixo embutido no model;
- compact notation precisa de teste para en/pt-BR e valores de borda;
- se o design exigir K/M como token visual invariável, documentar essa decisão separadamente da pontuação decimal; não usar `toStringAsFixed` como locale formatter.

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

Contrato V6:

- exceptions, PlatformException e MethodChannel usam código/reason estável como identidade;
- mensagem inglesa de exception não é copy de UI nem condição de lógica;
- `e.message`/`error.toString()` só pode chegar a UI se classificado como dado externo/diagnóstico e envolvido por contexto localizado;
- provider error text é third-party data, não tradução do produto;
- o mesmo reason code deve resolver em Flutter/native para significado equivalente;
- scanner rastreia sinks user-facing de mensagens cruas de erro.

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

## 6.8 Parsing de entrada humana locale-aware

A internacionalização precisa ser bidirecional: formatar **e** aceitar entrada.

Campos numéricos de produto que um usuário digita devem aceitar a convenção do locale efetivo e produzir valor canônico tipado.

Caso baseline confirmado:

    collection_editor_screen.dart
    rating 0..10
    pt-BR: 8,5
    en:    8.5

Regras:

- parsing usa locale explícito;
- não fazer replace global de vírgula/ponto sem validar agrupamento/ambiguidade;
- persistir número, não string localizada;
- validators usam mensagens ICU/localizadas;
- campos de IP/URL/porta/PIN/IDs continuam invariantes;
- testes cobrem colagem, teclado físico, IME e teclado TV quando a mesma entrada é alcançável.

Gate F deve detectar `double.parse/tryParse`, `num.parse` e regex decimal em input user-facing que contorne essa política.


## 6.9 Composição de listas e summaries

Não tratar toda ocorrência de `.join(...)` como bug. Classificar cada composição user-facing:

- **NATURAL_LANGUAGE_LIST** — nomes/itens que formam uma lista lida como frase; resolver com um `LocalizedListFormatter`/mensagem ICU que permita conjunção e padrão por locale;
- **VISUAL_METADATA_LIST** — blocos compactos como `ano · duração · gênero`; o separador pode ser design-invariant, mas cada item e sua ordem precisam de ownership/teste explícitos;
- **MULTILINE_STRUCTURED_LIST** — uma linha por item em dialog/report; newline pode ser invariant, porém cabeçalho, contagem e reason copy são localizados;
- **TECHNICAL_LIST** — IDs, codecs, URLs, tokens; não localizar automaticamente.

Casos baseline que entram no inventário:

- `onboarding/steps/done_step.dart`: services/trackers com `join(', ')`;
- `rewatch_progress_dialog.dart`: lista de falhas com `join(', ')` dentro de frase;
- `continue_watching_presentation.dart`, cards e metadata: joins com middle dot;
- Remote: labels falhos unidos por newline.

Regras:

- zero lista natural user-facing montada por separador inglês fixo sem classificação;
- para 0/1/2/3+ itens, testar output en/pt-BR;
- não traduzir os elementos quando forem brand/user/third-party data; localizar somente a gramática ao redor;
- não persistir o resultado já formatado quando a lista semântica puder ser persistida.

## 6.10 Grapheme-safe text handling

Sempre que a intenção for contar, truncar, pegar inicial, apagar ou limitar “caracteres” percebidos pelo usuário, operar por grapheme clusters, não por UTF-16 code units.

Inventariar caminhos user-facing com:

- `String.length` usado como limite visual;
- `substring`/índice `s[0]` para truncar/capitalizar nomes;
- helpers `truncate/capitalize/initials`;
- contadores/limites de input próprios;
- transformações que possam cortar combining marks, surrogate pairs ou sequências ZWJ.

Casos baseline confirmados:

- `pikpak_folder_picker_dialog.dart::_truncateFolderName` usa `name.length` + `substring`;
- `addon_hub_screen.dart::_capitalize` usa `s[0]` + `substring(1)`;
- `main.dart::_describeStartupFailure` trunca detalhe por code unit e deve ser classificado como diagnostic/external detail e tornado grapheme-safe se exibido;
- `addon_identity.dart` já usa `.characters.first`, mas o casing final ainda requer classificação.

Testar no mínimo:

- `Configuração`;
- forma decomposta `e\u0301`;
- emoji fora do BMP;
- emoji ZWJ/família;
- flag regional;
- string mista RTL/LTR quando relevante.

## 6.11 Casing e capitalização

Para `toUpperCase/toLowerCase` em presentation paths:

- tokens técnicos/design tokens podem manter transformação invariável se documentado;
- copy localizada deve preferir a forma fornecida pela tradução, sem transformação pós-l10n;
- dados externos/user data não devem ser “corrigidos” para uma convenção inglesa;
- helpers de capitalização devem ser grapheme-safe e não assumir que o primeiro code unit é uma letra inteira;
- se um locale futuro exigir case mapping especial, a política deve ser explícita e testável, sem espalhar transforms ad hoc.

## 6.12 Outbound human-text sinks

O scanner/inventário também considera texto humano produzido para fora de widgets:

- `ClipboardData(text: ...)` quando o conteúdo é copy do produto, não apenas URL/token/user data;
- templates de compartilhamento quando existirem;
- texto de export/report criado para leitura humana;
- nomes/títulos sugeridos a FilePicker/chooser/plugins;
- content descriptions/titles enviados a APIs de sistema;
- bodies de notificação e outras superfícies já classificadas.

Cada ocorrência recebe ownership: PRODUCT_COPY, USER_DATA, THIRD_PARTY_DATA, BRAND/TECHNICAL ou DIAGNOSTIC_DETAIL. Um sink invisível ao widget tree não pode ser automaticamente excluído da auditoria.

## 6.13 Taxonomia temporal obrigatória

Antes de migrar qualquer data/hora, classificar a intenção:

| Classe | Exemplo | Política |
|---|---|---|
| `HUMAN_DATE` | expiração, calendário, last updated | formatter com locale efetivo |
| `CIVIL_TIME` | horário de programa/agendamento | locale + preferência 12/24h quando aplicável |
| `PRODUCT_FIXED_CLOCK` | grade que deliberadamente usa 24h | permitido somente com justificativa UX explícita |
| `MEDIA_TIMECODE` | `01:23:45` do player | formato funcional/invariante; não aplicar NumberFormat por dígito |
| `PROTOCOL_DATE_TIME` | API/schema/Trakt path | exatamente o formato do protocolo |
| `FILENAME_TIMESTAMP` | nome de backup/recording | estável/filesystem-safe |
| `DIAGNOSTIC_TIMESTAMP` | ISO UTC de log | técnico/invariante |
| `PROVIDER_CALENDAR_RULE` | chunk Trakt começando segunda | regra funcional do provider, não preferência visual |

Regras:

- não transformar `MEDIA_TIMECODE` em hora civil;
- não alterar API/cache chunking porque o primeiro dia visual da semana mudou;
- UI de calendário pode usar first-day-of-week do locale/framework sem reescrever o algoritmo de fetch do provider;
- `alwaysUse24HourFormat: true` em UI humana exige classificação/justificativa;
- montagem manual de weekdays/months/day/month order em presentation é finding;
- montagem técnica de timestamp/path só é finding se classificada incorretamente como UI.

## 6.14 Fronteira de formatação: domain retorna dados, presentation retorna texto

Models/services não devem expor como API principal:

- `formattedExpiration`;
- `formattedSize`;
- `formattedDate`;
- `N/A` como substituto textual de ausência;
- frases de relative time.

Preferir:

    DateTime? expiration
    int bytes
    Duration age
    enum/status nullable

e formatar na borda com `LocalizedFormatters` + AppLocalizations.

Exceção: serialização/protocolo pode gerar string canônica, mas o nome/API deve deixar claro que é formato técnico e não presentation copy.

## 6.15 Collation e ordenação visual

Para lista controlada pelo produto:

- manter ordem semanticamente definida por padrão;
- se “ordem alfabética no idioma ativo” for requisito, usar solução de collation real e testada;
- `String.compareTo`, `toLowerCase`, remoção de acentos e search-fold **não** qualificam como collation locale-aware;
- provider/user data mantém a ordenação definida pela fonte salvo requisito explícito;
- mudar App language não pode reordenar dados cuja ordenação é estado persistido/semântico, a menos que a UI declare ser uma ordenação puramente visual.


## 6.16 Compact numbers e métricas sociais

Formatação compacta de contagem é apresentação locale-sensitive.

Inventariar getters/helpers como:

- `formattedViews`;
- `formattedScore`;
- `formattedCount`;
- `toStringAsFixed(...) + 'K/M/B'`;
- número + substantivo inglês (`views`, `votes`, `comments`).

O valor canônico permanece `int/double`. A borda de apresentação decide compactação e gramática.

Testes mínimos:

- 999;
- 1.000;
- 1.250;
- 999.999;
- 1.000.000;
- zero/null quando aplicável;
- en x pt-BR;
- sem perda de identidade/valor usado em sort/filter/API.

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

## 7.5 Semantics e language attribution

Localizar:

- semanticLabel;
- button descriptions;
- tooltips;
- contentDescription nativo;
- status de progresso;
- toggles;
- image semantics quando realmente necessárias.

Quando App language divergir do system locale, validar também a **língua atribuída à árvore Semantics**, não somente o texto:

- no Flutter 3.44.8, a implementação de `Localizations` application-level não fornece `localeForSubtree` nesse wrapper;
- criar teste com `SemanticsTester`/inspeção equivalente para verificar o locale efetivamente exposto ao subtree;
- se o root não propagar o override de forma suficiente, aplicar `Semantics(localeForSubtree: effectiveLocale)` **somente a scopes cujo texto semântico é app-owned por contrato**;
- não usar um wrapper global cego quando a subtree inclui title/filename/URL/user data/third-party copy em outro idioma;
- em linha mista, separar semantic nodes quando necessário: chrome app-owned recebe App language; external/user data mantém language metadata própria quando confiável ou permanece sem atribuição inventada;
- não atribuir artificialmente o App language a trechos que são deliberadamente external/user data em outro idioma quando houver semântica mais adequada;
- screen reader real continua no Gate O porque pronúncia/voz final também depende da plataforma e das vozes instaladas.

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

### V6 — sinks do teclado próprio

O teclado Debrify é UI localizada, não apenas mecanismo de input. Submit/action labels, accessibility e instruções pertencem ao locale efetivo do app. O overlay deve receber/resolver copy do mesmo AppLocaleController mesmo quando criado fora da subtree visual imediata.

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
- hints;
- PiP `RemoteAction` title/contentDescription (`Play`, `Pause`, `Next` na baseline);
- copy mista de accessibility como `${externalData}, Playing`, convertida para formatted resource.

Cobrir no mínimo:

- AndroidTvTorrentPlayerActivity;
- TorboxTvPlayerActivity;
- MainActivity quando exibe copy, inclusive PiP `RemoteAction`;
- qualquer API OS-surfaced em que o app forneça title/description/action label;
- controllers/helpers nativos utilizados pelos players.

## Flutter -> native locale contract

O locale escolhido no Flutter deve ser transferido explicitamente para a camada nativa.

Contrato:

    uiLocale = system | ProductLocaleId

Centralizar em `NativeLocaleBridge` + `NativeLocaleStore`.

### NativeLocaleStore

A V3 **não cria uma segunda preferência/mirror** quando a mesma preferência device-owned pode ser lida por Dart e Android nativo.

Fonte persistida única:

    ui_locale_v1 = system | ProductLocaleId

Dart:

- lê/escreve por `DevicePreferences`;
- registra a key em `DevicePreferences.allowedKeys`;
- valida contra ShippingLocales antes da gravação.

Android nativo:

- `NativeLocaleStore` é um adapter de leitura da preferência física escrita pelo plugin;
- a baseline usa `shared_preferences 2.5.3` / `shared_preferences_android 2.4.10`, `DevicePreferences` chama o `SharedPreferences.getInstance()` legado e o upstream já lê esse store nativamente; portanto o contrato físico auditado é **arquivo `FlutterSharedPreferences`, key `flutter.ui_locale_v1`**;
- esse detalhe fica isolado em um único adapter;
- contract test fixa o backing file/key e versões resolvidas do plugin; Dart escreve `ui_locale_v1` e um reader nativo cold-start deve observar `flutter.ui_locale_v1` no mesmo store;
- se o backend/API do plugin mudar (por exemplo migração futura para outro backend), Gate 0/Gate Q bloqueiam promoção até o adapter/contract test ser atualizado; não criar mirror para esconder incompatibilidade.

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

## Android system per-app languages — decisão V9

Na baseline auditada não existem `android:localeConfig`, `LocaleManager`, `AppCompatDelegate.setApplicationLocales` nem `generateLocaleConfig`.

**Política do primeiro release pt-BR:** não introduzir Android per-app language como segunda autoridade. A autoridade do produto continua:

    DevicePreferences/ui_locale_v1
      -> AppLocaleController
      -> NativeLocaleBridge/NativeLocaleStore

O seletor interno do Debrify permanece canônico em Android/Android TV.

Se uma versão futura quiser integrar o seletor do sistema Android:

- fazer em PR arquitetural dedicado;
- definir qual storage é canônico e como migra `ui_locale_v1`;
- definir reconciliação quando o SO e o store divergem;
- provar cold-start/background/process-kill;
- provar que Activities nativas e Flutter recebem o mesmo locale;
- provar continuidade de playback/focus em Android TV;
- provar backup/restore/reset;
- eliminar, e não duplicar, autoridades.

Gate H falha se APIs/configs de per-app language forem introduzidas sem esse contrato de migração.

### APP_SUPPLIED_SYSTEM_UI no Android e plugins

Quando o app fornece texto que o SO/plugin desenha, esse texto continua sendo responsabilidade i18n do Debrify.

Casos baseline:

- `MainActivity.buildPipActions()` -> `RemoteAction` Play/Pause/Next;
- `FilePicker.platform.pickFiles/saveFile(dialogTitle: ...)` em Settings, Profiles, Remote e Collections;
- notification action labels/titles;
- qualquer chooser/dialog/plugin com title/label passado pelo app.

O chrome realmente pertencente ao SO pode continuar no idioma do sistema. O teste deve aceitar essa fronteira e verificar apenas a copy controlada pelo Debrify; não prometer que um override interno muda botões nativos que a API não permite controlar.

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

A baseline possui **duas fontes divergentes** de `.desktop`:

1. `linux/debrify.desktop` com `Comment=Debrid client for torrents and streaming`;
2. `.github/workflows/build.yml`, nos jobs Linux x64 e arm64, gera inline `AppDir/debrify.desktop` com `Comment=Modern debrid companion with torrent search and playlist management`.

Isso significa que localizar só o arquivo versionado não altera o AppImage publicado.

Requisitos:

- definir uma única fonte canônica de `.desktop` usada por x64 e arm64 (template/script ou arquivo versionado parametrizado);
- `Name=Debrify` permanece brand;
- adicionar localized key FreeDesktop `Comment[pt_BR]=...` para shipping PT-BR;
- auditar `GenericName`, `Keywords`, Actions e outros campos localizáveis se surgirem;
- manter `Exec`, `MimeType`, `Categories`, `X-AppImage-*`, protocols e IDs como dados técnicos;
- o locale do `.desktop` é escolhido pelo desktop environment/OS, não pelo override Dart em runtime;
- scanner inclui `.desktop`, workflows e scripts que geram metadata de shell;
- Gate N extrai/inspeciona AppImages x86_64 e arm64 e confirma a entrada `pt_BR` no `.desktop` realmente empacotado.

O window title Debrify pode permanecer marca. Qualquer outra mensagem GTK/native user-facing deve entrar no catálogo ou ser classificada/allowlisted.

# 8.7 Web

Corrigir primeiro a qualidade base:

- remover A new Flutter project do manifest e index.html;
- usar nome/descrição canônicos do Debrify;
- revisar title;
- revisar apple-mobile-web-app-title;
- revisar lang do documento quando viável;
- garantir que o shell não prometa idioma diferente do app;
- PWA metadata não pode ficar como placeholder de projeto Flutter.

**Fronteira de autoridade Web/PWA:** `documentElement.lang/dir` pode acompanhar o App language em runtime; `manifest.json` e metadata de instalação são recursos estáticos/cacheados pelo browser. Para campos language-sensitive do manifest, escolher explicitamente metadata neutra/brand-only ou manifest/HTML locale-aware servido por URL/negociação inicial, com teste de cache/install.

Não alterar `manifest.lang` em memória e presumir que o metadata de um PWA já instalado mudou. `lang` do manifest descreve o idioma da metadata do manifest; não é a autoridade do AppLocaleController.

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
- justificativa se allowlisted;
- `origin` (`SOURCE_FILE`, `BUILD_GENERATED`, `REMOTE`, `SYSTEM`);
- artifact/targets alcançados quando aplicável;
- owner da copy;
- locale authority/source aplicável.

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
- BRAND_ART_DIRECTION;
- APP_SUPPLIED_SYSTEM_UI;
- BUILD_GENERATED_PRODUCT_COPY.

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
- notification copy configurada por plugins;
- `FilePicker`/plugin dialog titles e outras strings app-supplied em superfícies do SO;
- Android `RemoteAction`/PiP e outras ações OS-surfaced;
- `.desktop`/AppImage metadata;
- `.github/workflows` e scripts/templates que gerem user-facing artifact metadata;
- Android resources em `string`, `plurals`, `string-array`, menus e atributos accessibility/string-bearing, não apenas layouts.

## 9.1 Escopo por reachability do artefato

A V5 substitui a métrica histórica ambígua por contagens reproduzíveis da árvore Git `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`:

- 3.199 blobs/arquivos no repositório;
- 1.736 arquivos nos roots first-party de produto `lib/android/assets/ios/linux/macos/tvos/web/windows`;
- 2.109 incluindo `packages/`.

Essas contagens medem estrutura, não copy user-facing. O inventory decide reachability/classificação por arquivo e sink.

O scanner deve:

1. incluir todo código próprio que chega aos artifacts publicados;
2. incluir `packages/` locais/vendorizados quando o package participa do runtime;
3. classificar exemplos/testes/dev/generated separadamente;
4. nunca excluir `packages/` ou `dev/` apenas por nome de pasta sem provar que não chegam ao artifact;
5. permitir allowlist de terceiro somente com path, reason, owner e revisão;
6. incluir build-time provenance: workflow/script/template que produz arquivo user-facing no artifact também é reachable;
7. não assumir que um arquivo versionado é a fonte realmente empacotada; comparar source-of-truth com conteúdo final do artifact.

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

## 9.2 Blind spots adicionados pela V6

O inventário/gate passa a reconhecer explicitamente:

- tabelas/mapas `language code -> English display name`;
- `keyboardSubmitLabel`, `suggestionsLabel` e action rows de teclado próprio;
- `PlatformException.message`, `Exception.toString()`, `e.message` e raw remote/provider messages quando chegam a UI;
- campos/protocolos cross-device que carregam copy humana;
- parsers de número ligados a TextField/TvTextField/validators;
- reason/result codes sem catálogo de apresentação;
- stale/orphan ARB keys e allowlist entries sem sink/finding correspondente.

Cada finding deve distinguir dado externo, token técnico, identidade semântica e copy localizável.

## 9.3 Runtime packages — regra operacional de scan

Além dos roots principais, os scanners percorrem recursivamente, quando existentes:

    packages/**/lib/**
    packages/**/android/**
    packages/**/ios/**
    packages/**/tvos/**
    packages/**/macos/**
    packages/**/linux/**
    packages/**/windows/**
    packages/**/web/**

Cada package alcançável no artifact recebe uma classe:

- `FIRST_PARTY_FORK` — copy/control flow alterados pelo Debrify entram integralmente nos gates;
- `VENDORED_THIRD_PARTY` — upstream third-party, mas qualquer app-owned patch/copy precisa de ownership;
- `GENERATED` — validar a fonte geradora/artifact, não editar o resultado à mão.

“Está em packages/” nunca é exclusão automática.


## 9.4 Prova de fechamento do universo auditado

Alegar cobertura total exige uma relação verificável com a Git tree, não apenas “scanner retornou zero findings”.

Definir:

    RuntimeSurfaceUniverse
      = todos os blobs/paths da baseline que podem participar de artifact/runtime/product copy

    ClassifiedReachable
      = paths analisados e classificados com ownership + sinks

    ExcludedWithEvidence
      = paths provadamente fora de runtime/product surface, com reason

Invariante obrigatório:

    RuntimeSurfaceUniverse
      == ClassifiedReachable ∪ ExcludedWithEvidence

e:

    ClassifiedReachable ∩ ExcludedWithEvidence == ∅

CI/Gate Q falha com:

- path novo sem classificação;
- path removido ainda presente no manifest;
- glob que cobre arquivos novos sem revisão quando o tipo do path pode carregar UI;
- arquivo classificado como generated sem fonte geradora registrada;
- arquivo vendorizado modificado pelo Debrify sem reclassificação;
- diferença entre tree auditada e universo registrado.

O inventory deve registrar pelo menos:

- path;
- blob SHA quando a baseline é fixada;
- reachability;
- ownership;
- source kind;
- parser/scanner aplicável;
- finding count/status;
- exclusion reason quando aplicável.

Isso transforma “100%” em propriedade auditável da baseline, não em afirmação manual.

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
- baseline de hardcoded directionality com classificação semântica/física;
- inventário de `APP_SUPPLIED_SYSTEM_UI` (PiP, FilePicker/dialogTitle, plugin/system surfaces);
- inventário de `BUILD_GENERATED_PRODUCT_COPY`, inclusive `.github/workflows/build.yml` -> AppImage `.desktop`;
- baseline dos test harnesses: MaterialApp/Directionality ad hoc e asserts textuais em inglês.
- importar `docs/AUDIT_HOTSPOTS_V10.json` como seed verificável (herdando V9), sem tratá-lo como inventário completo;
- classificar runtime packages alcançáveis em FIRST_PARTY_FORK/VENDORED_THIRD_PARTY/GENERATED;
- baseline de calendar/date tables, manual date assembly, fixed-hour-cycle e model-level presentation formatting.

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
- helper/renderer reorder-safe para rich text localizado;
- `AppLocalizations.localizationsDelegates`/delegates Material+Widgets+Cupertino em todos os roots;
- `LocalizedCopyResolver` (ou equivalente) para copy fora de BuildContext sem criar segunda autoridade;
- `localizedTestApp`/harness comum para testes que renderizam UI localizada.
- `LocaleFallbackPolicy` versionada e separada de ShippingLocales;
- `localeEpoch`/LocaleSnapshot para impedir async presentation commit stale;
- source guard inicial contra segunda autoridade Android per-app language.

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
- locale flip preserva query e input/form state;
- completion assíncrona de Settings Search/support config do locale anterior não pode sobrescrever o locale atual.

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
- app-supplied file picker dialog titles (classificação `APP_SUPPLIED_SYSTEM_UI`);
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
- locale flip não reinicia playback, não troca source e não perde posição/session; qualquer recreation nativa futura precisa de handoff explícito.

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
- notifications nativas relacionadas;
- `MainActivity` PiP `RemoteAction` Play/Pause/Next;
- accessibility strings nativas com placeholders de external data e bidi safety.

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
- remover weekdays/months/Today/Tomorrow/Yesterday manuais dos paths shipping;
- mover formatter de apresentação para a borda quando hoje estiver em model/service;
- preservar `TraktCalendarService` Monday-aligned chunking como PROVIDER_CALENDAR_RULE, separado do primeiro dia visual da semana.

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
- ownership/limites de locale OS-managed em Apple/installer;
- Linux AppImage x64/arm64 usando uma única fonte `.desktop` locale-aware, sem copy inglesa inline divergente no workflow;
- PWA manifest/static install metadata com política separada do DOM runtime;
- Windows protocol handler: IDs técnicos preservados e descrições humanas classificadas/localizadas;
- inspeção pós-build de recursos/localizações dentro de cada artifact.

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
- taxonomia temporal HUMAN_DATE/CIVIL_TIME/FIXED_CLOCK/TIMECODE/PROTOCOL/FILENAME/DIAGNOSTIC/PROVIDER_RULE;
- Unicode search conformance NFC/NFD/combining marks;
- collation real vs search-fold;
- mixed-language Semantics scoped por ownership;
- locale-switch state continuity.

## Fase 13 — Zero-regression cleanup

Fechar antes da promoção PT-BR os hardenings V6:

- consolidar display names de idiomas por identidade canônica;
- eliminar parsing decimal humano invariável em inputs locale-sensitive;
- localizar todos os sinks do teclado próprio de TV;
- substituir raw human error messages em boundaries por reason codes;
- migrar Remote cross-device para result/reason codes com compatibilidade legada;
- executar a matriz de runtime real definida pelo Gate O;
- executar completions assíncronas fora de ordem contra `localeEpoch`;
- executar locale flip en → pt-BR → en com navigation/player/download/recording/pairing/form state vivos;
- executar scanner recursivo nos runtime roots de `packages/**`;

Objetivo:

- hardcoded UI baseline -> zero não justificado;
- semantic coupling -> zero;
- shipping locale completeness -> 100%;
- placeholder parity -> 100%;
- native resource parity -> 100%;
- todos os allowlists com justificativa e owner;
- test harness localizado consolidado e asserts de comportamento desacoplados de inglês onde texto não é o objeto do teste;
- artifact-localization report verde antes de promoção.

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

## 11.0 Registro canônico de gates

Os IDs abaixo são API documental estável. `PLANO_MESTRE_V10.md`, `docs/CI_QUALITY_GATES.md`, matriz de testes, PR templates e scripts devem usar **os mesmos IDs**; aliases numéricos ou históricos são proibidos.

| ID | Contrato |
|---|---|
| 0 | dependency/toolchain preflight |
| A | geração/toolchain |
| B | ARB schema/parity/lifecycle |
| C | ShippingLocales |
| D | hardcoded/reachability/product text |
| E | semantic coupling + presentation-cache identity |
| F | formatters + localized human input |
| G | Flutter analyze/test |
| H | Android native |
| I | platform build matrix |
| J | pseudo/layout |
| K | secrets/log privacy |
| L | remote/runtime/official product copy |
| M | directionality/inline-text safety |
| N | packaged artifact localization |
| O | real-device runtime localization smoke |
| P | Unicode/composition/outbound text sinks |
| Q | upstream baseline drift + audit freshness |

Nenhum gate é “opcional por ter outro número”. Um documento auxiliar que divergir deste registro falha Gate Q/document-consistency.

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
- untranslated shipping messages;
- keys ARB órfãs/não alcançadas, salvo allowlist explícita;
- metadata/descrições obrigatórias semânticas para novas keys;
- allowlist stale que já não corresponde a finding real.

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
- visual runtime assets com texto embutido (SVG estrutural + revisão manual para raster/PDF);
- FilePicker/plugin `dialogTitle` e outras strings app-supplied em UI do sistema;
- Android PiP/`RemoteAction` title/contentDescription;
- `.desktop`, workflow YAML, shell/CMake/ISS/templates que geram metadata user-facing no artifact;
- Android `string-array`, menus e demais resource XML string-bearing;
- `translatable=false` usado em user-facing resource sem justificativa;
- `keyboardSubmitLabel`, `suggestionsLabel` e action labels do teclado Debrify;
- mapas/tabelas próprios que convertam language code diretamente para nome inglês user-facing;
- raw exception/provider/remote `message` chegando a Text/SnackBar/Dialog sem classificação.

## Gate E — Semantic coupling e presentation-cache identity

Falhar em novos casos de:

- == 'display text';
- switch em copy;
- id derivado de label;
- persistence de localized string;
- lógica baseada em tradução;
- protocolo cross-device que usa texto localizado como result/status identity;
- UI que decide comportamento por `PlatformException.message`/`Exception.toString()`;
- singleton/static/cache que guarda copy localizada capaz de sobreviver à troca de idioma sem `ProductLocaleId`/invalidation;
- resultado já localizado tratado como estado canônico em vez de semantic state + arguments;
- async locale-sensitive result que publica copy sem validar `localeEpoch`/snapshot atual;
- troca de locale que reseta navigation/playback/download/recording/pairing/form input em vez de apenas presentation state.

## Gate F — Formatters

Testar ao menos:

- en;
- pt_BR;
- timezone;
- decimal;
- plural;
- date/time;
- relative time;
- parsing de número humano en/pt-BR;
- separação entre número humano e token técnico;
- round-trip entrada localizada -> valor canônico -> apresentação localizada;
- weekdays/months/Today/Tomorrow/Yesterday montados manualmente em presentation;
- `DateFormat`/`NumberFormat` user-facing sem locale efetivo explícito;
- datas humanas montadas por `${year}-${month}`/`${day}/${month}` sem classificação;
- `alwaysUse24HourFormat: true` classificado como `PRODUCT_FIXED_CLOCK` ou corrigido para preferência civil;
- `MEDIA_TIMECODE`, `PROTOCOL_DATE_TIME`, `FILENAME_TIMESTAMP` e `DIAGNOSTIC_TIMESTAMP` preservados como invariantes quando apropriado;
- first-day-of-week visual separado de `PROVIDER_CALENDAR_RULE`;
- nenhuma alegação de collation locale-aware implementada somente com `String.compareTo`/lowercase/search-fold.

## Gate G — Flutter analysis/test

    flutter analyze
    flutter test

Durante migração podem existir suites segmentadas por fase, mas antes da promoção PT-BR a suite oficial completa deve estar verde ou qualquer falha preexistente deve estar registrada e reproduzida no baseline.

Adicionar harness comum (`localizedTestApp` ou equivalente) com delegates, ShippingLocales e locale explícito. Testes cujo objetivo não é wording devem preferir keys/semantics/widget identity a `find.text('English literal')`; testes de localização devem assertar a mensagem gerada para o locale sob teste. Migrar MaterialApp/Directionality ad hoc à medida que cada domínio entra em i18n.

## Gate H — Android native

Executar unit/Robolectric/Gradle pertinentes.

Verificar:

- resource parity entre values e values-pt-rBR por nome **e tipo**;
- parity de placeholders/formats (`%1$s`, `%1$d`, plurals, positional arguments, `formatted`);
- recursos `string-array`/menu/accessibility quando user-facing;
- zero literal user-facing em layouts XML fora de allowlist;
- ambas Activities de player;
- `MainActivity` PiP `RemoteAction` e atualização de actions no locale efetivo;
- NativeLocaleStore;
- contract físico `FlutterSharedPreferences / flutter.ui_locale_v1` no baseline resolvido;
- Dart write → native cold-read e valor físico corrompido → fallback seguro;
- Services/Receivers com processo Flutter morto;
- notifications, actions, plurals e channels;
- ausência de branch baseado em localized/display text;
- ausência de `android:localeConfig`/LocaleManager/AppCompat per-app-language introduzidos sem PR/contrato de migração de autoridade V9;
- locale flip em Activity/player preserva sessão/posição/focus conforme a superfície exigir.

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
- desktop;
- Semantics/contentDescription;
- Semantics locale attribution quando App language != system locale;
- mixed-language semantics: app-owned copy recebe App language sem rotular cegamente external/user data;
- locale flip com tela viva preserva navigation state, foco TV e input não salvo.

## Gate K — No secrets/log regression

Mudança de erros/localização não deve colocar tokens, URLs sensíveis, credentials ou payloads privados em mensagens de UI/log.

## Gate L — Remote/runtime product copy

Falhar ou registrar blocker quando:

- campo user-facing oficialmente controlado pelo produto não possui política de locale;
- fallback asset/remote config contém copy própria fora do contrato;
- shipping locale cai para en em `REMOTE_PRODUCT_COPY`;
- catálogo remoto oficial exibe descrição editorial sem locale/mapeamento aprovado;
- cache de remote copy pode servir variante do locale anterior;
- runtime asset user-facing escapa do inventário;
- conteúdo oficial essencial acionado pelo app não declara a fronteira App language ↔ browser/system locale;
- link oficial externo assume que `Accept-Language` do browser representa o override interno;
- QR aberto em outro dispositivo é tratado como se herdasse o App language do dispositivo de origem.

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


## Gate N — Packaged artifact localization

Source code verde não basta. Para cada artifact publicado, gerar relatório pós-build que prove que a localização chegou ao pacote final.

Verificar, conforme plataforma:

- Android APK: resources `values-pt-rBR`, strings/plurals usados pelos players/services/PiP após shrink/minify;
- iOS/tvOS IPA: `pt-BR.lproj`, `InfoPlist.strings`, target membership e TopShelf extension quando aplicável;
- macOS `.app`/DMG: `pt-BR.lproj`, InfoPlist/MainMenu localization;
- Windows installer: BrazilianPortuguese presente, CustomMessages/tasks/run text localizados, protocol IDs intactos;
- Linux x86_64/arm64 AppImage: `.desktop` final contém `Comment[pt_BR]` e não reintroduz English-only comment por geração inline;
- Web build, quando suportado/publicado: shell sem placeholder e política de manifest/HTML verificada.

O gate deve também falhar se uma source canônica localizada for sobrescrita posteriormente pelo workflow com copy não localizada.

---
## Gate O — Real-device runtime localization smoke

Gate de promoção para superfícies que source/artifact inspection não consegue provar sozinha.

Registrar, com artifact SHA/build e evidência:

- Android phone/tablet: system en + App language pt-BR e system pt-BR + App language en;
- Android TV em hardware representativo: ambos players nativos, D-pad/focus, teclado próprio, notifications/channels e PiP quando suportado;
- tvOS em Apple TV real/suportada: focus/input, Top Shelf e fronteira system/app language;
- iOS/macOS/Windows/Linux: smoke das superfícies nativas publicadas quando aplicável;
- screen reader/accessibility em pelo menos um cenário system language != App language;
- inspeção prova que a subtree Flutter expõe o locale do App language onde a copy é app-owned;
- locale flip durante playback/download/formulário ativo preserva estado funcional e invalida somente presentation state;
- pelo menos um cenário força completion assíncrona antiga após a troca de locale e prova que ela não sobrescreve copy nova.

Aceite:

- copy controlada pelo Debrify usa App language quando a plataforma permite;
- chrome/prompts realmente OS-owned seguem a autoridade documentada do SO;
- nenhuma superfície crítica mistura inglês por copy app-owned não inventariada;
- evidência visual/log de cada exceção OS-owned é anexada ao release audit.

Este gate não exige fingir controle sobre UI que pertence ao SO; ele prova justamente a fronteira real.

## Gate P — Unicode, composition e outbound text sinks

Falhar ou exigir allowlist versionada quando um path user-facing possuir:

- lista natural criada por `.join(', ')`/separador fixo sem formatter/ICU/classificação;
- helper visual de truncate/capitalize/initials baseado em `String.length`, `substring`, `s[0]` ou code unit;
- casing pós-localização não classificado;
- Clipboard/share/export/report/plugin/system human-text sink sem ownership;
- localized string persistida como resultado de composição quando dados semânticos poderiam ser persistidos;
- cache de apresentação que guarda copy localizada sem `ProductLocaleId`/invalidation quando a copy pode sobreviver à troca de idioma.

Exigir testes para:

- listas 0/1/2/3+ em en e pt-BR;
- combining marks;
- non-BMP;
- ZWJ emoji;
- regional flags;
- locale flip en → pt-BR → en sem restart para helpers/caches long-lived;
- equivalência NFC/NFD/combining marks no `SettingsSearchNormalizer` para PT-BR;
- async locale result fora de ordem protegido por epoch;
- mixed-script/mixed-language semantic attribution quando aplicável.

## Gate Q — Upstream baseline drift e audit freshness

A execução nunca presume que a árvore continua igual ao dia da auditoria.

Fonte machine-readable:

    docs/AUDIT_BASELINE_MANIFEST.json

Antes de iniciar a implementação, antes de cada rebase relevante e obrigatoriamente antes de promoção/release:

1. resolver o SHA upstream que será a nova base;
2. comparar com `auditedCommit` do manifest;
3. se igual, registrar evidência e prosseguir;
4. se diferente, obter `git diff --name-status auditedCommit..newBase`/equivalente;
5. classificar cada path alterado por reachability/ownership;
6. rerodar scanners D–P nos paths alterados e, na promoção, o scan completo;
7. atualizar inventário, allowlists, contagens estruturais quando aplicável e manifest somente após revisão;
8. registrar o novo tree SHA e provar que a API respondeu `truncated=false` ou usar paginação/estratégia equivalente completa.

Bloquear quando:

- upstream SHA mudou sem audit delta;
- arquivo/target/runtime asset/build script novo não possui classificação;
- allowlist foi carregada de um baseline diferente sem revalidação;
- versão de Flutter, `shared_preferences`, gen_l10n, workflow de packaging ou plataforma mudou e seus contratos dependentes não foram revalidados;
- documentos normativos usam gate IDs diferentes do registro canônico atual;
- novo/alterado root runtime sob `packages/**` não foi classificado/escaneado;
- mudança em calendário/formatters/platform locale APIs não foi reclassificada pela taxonomia V9.

Mudanças da própria branch de implementação não são “drift do upstream”; o gate compara a **base upstream auditada** com a nova base upstream pretendida.


### Reforço V10 — Gate D — indirect/declarative presentation dataflow

Além dos sinks diretos, reportar PRODUCT_COPY em origens indiretas que alcançam UI:

- enum constructor fields;
- extension/model/service getters `label/title/subtitle/description/displayName/statusText/*Text/formatted*`;
- registries/options/records/static tables;
- presentation strings passadas por múltiplas camadas antes do widget.

A análise pode combinar AST + symbol/name heuristics + sink tracing. Heurística gera finding para classificação; não transforma automaticamente brand/external/user data em tradução.

### Reforço V10 — Gate E — identity/display separation

Falhar quando:

- option persistida usa label localizado em vez de stable id;
- Settings Search usa uma copy diferente da tela para a mesma option sem política explícita;
- getter de domínio materializa PRODUCT_COPY que deveria ser resolvida na presentation layer;
- locale flip exige reconstruir estado semântico apenas porque o model armazenou presentation string.

### Reforço V10 — Gate F — temporal/compact-pattern detection

Reportar/classificar em presentation paths:

- `hour % 12` / `hour >= 12` + `AM/PM`;
- relógio civil por `hour/minute.padLeft`;
- `toStringAsFixed` usado como formatter de número humano;
- compactação manual `K/M/B`;
- número concatenado com substantivo app-owned;
- `formatted*/progressText/statusText` em models/services quando retorna data/hora/número/frase final.

### Reforço V10 — Gate P — declarative copy lifecycle

Unicode/composition scan também alcança strings vindas de registry/model mapper; não considerar seguro apenas porque a copy não está literal no widget.

### Reforço V10 — Gate Q — coverage closure

Além de SHA/tree drift, validar a igualdade de conjuntos definida em 9.4. Nenhum blob runtime relevante pode existir fora do ledger de cobertura; nenhuma exclusão pode permanecer sem path/reason/evidence vigente.

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
- `zh-Hant-TW` / `sr-Latn-RS` preservam script+region quando vierem a ser generated/test locales;
- `en-US-u-hc-h12`, variant/private-use e tags malformed não são persistidas por perda silenciosa;
- `system` permanece sentinel e nunca é confundido com language tag;
- unknown language;
- system locale changes while followsSystem;
- system change ignored while manual override;
- restart;
- Web DOM lang/dir update;
- app locale vs OS-owned locale boundary;
- framework Material/Widgets/Cupertino delegates presentes em todos os roots;
- `showDatePicker`/`showTimePicker` usam pt-BR sob override pt-BR.

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
- PiP `RemoteAction` Play/Pause/Next em en/PT-BR;
- FilePicker `dialogTitle` app-supplied em en/PT-BR onde a plataforma expõe o título;
- native mixed-data semantics com placeholder/bidi safety;
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
- Linux `.desktop` versionado e `.desktop` efetivamente gerado dentro dos AppImages x86_64/arm64;
- packaged-artifact localization report;
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

## Language display names

- `eng/spa/por/pt-BR/por-br` resolvem identidade consistente;
- en e pt-BR exibem nomes próprios localizados;
- unknown code mantém fallback técnico previsível;
- external provider label não é traduzido por engano;
- Dart/native Android possuem paridade semântica.

## Localized input parsing

- pt-BR aceita `8,5` em rating humano;
- en aceita `8.5`;
- valor persistido é numérico/canônico;
- input inválido/ambíguo é rejeitado com copy localizada;
- IP `100.64.0.5`, URL, porta, PIN e schema date continuam invariantes.

## Cross-runtime errors

- mesmo reason code resolve copy en/pt-BR;
- `PlatformException.message` não é requisito para UX;
- erro de provider externo aparece apenas como detalhe classificado;
- ausência/mensagem malformada não quebra fallback localizado.

## Remote cross-device locale

Executar combinações:

    sender en    -> receiver pt-BR
    sender pt-BR -> receiver en

Validar pairing, transfer success/failure, permission/admin blockers e retry. O texto final sempre pertence ao locale do runtime que o mostra.

## TV keyboard

- defaults Search/Go/Next/Send/Done localizados;
- custom `keyboardSubmitLabel` localizado;
- Clear/paste/voice/backspace semantics localizados;
- `?123`/ABC classificados como design/technical quando mantidos;
- overlay não perde locale ao abrir por portal/OverlayEntry.

## Real-device smoke

Executar o Gate O e anexar artifact SHA, system locale, App language, dispositivo/OS e screenshot/gravação por cenário crítico.

## Calendar/temporal semantics V9

Testar, no mínimo:

- `trakt_calendar_day_sheet`: data completa en/pt-BR sem tabela inglesa manual;
- `trakt_calendar_screen`: headline/month/short weekday en/pt-BR;
- `iptv_epg_panel`: hoje/amanhã/ontem + dia/mês;
- recordings: data humana pt-BR e picker framework-localized;
- system 12h + App language en: civil time respeita 12h quando a superfície não é fixed-clock;
- system 24h + App language pt-BR: civil time respeita 24h;
- media timecode permanece `mm:ss`/`h:mm:ss` e não ganha separadores decimais/localized digits por engano;
- Trakt API chunk continua Monday-aligned quando esse é o contrato do serviço, independentemente do primeiro dia visual da semana;
- `RdUser`/debrid models expõem dados canônicos e presentation formata expiração/size;
- locale flip não altera timestamp técnico/path/filename/API serialization.

## Async locale epoch V9

Harness com completers controlados:

1. locale en, iniciar remote/presentation load A;
2. trocar para pt-BR, epoch++;
3. iniciar load B;
4. concluir B e renderizar pt-BR;
5. concluir A atrasado;
6. provar que UI/cache materializado continua pt-BR;
7. repetir pt-BR -> en;
8. provar que semantic payload neutro ainda pode ser reaproveitado sem copy stale.

## Resolution/fallback stability V9

- `pt -> pt-BR` vem da policy explícita;
- `pt-PT` não cai em pt-BR;
- preferred list `[pt-PT, en-US]` alcança en quando pt-PT não possui match permitido;
- simular inclusão futura de pt-PT e provar que nenhum default muda implicitamente;
- policy version change exige teste de upgrade.

## Mixed-language accessibility V9

- App language pt-BR + media title inglês;
- App language en + filename/título português;
- localized chrome recebe locale semântico do app;
- external/user data não recebe language tag inventada;
- nested trusted-language metadata, quando existir, prevalece apenas no subtree apropriado.

## Locale-switch continuity V9

Com tela viva, alternar en ↔ pt-BR e provar, conforme superfície:

- navigation stack preservado;
- Settings Search/query e formulário não salvo preservados;
- D-pad focus permanece semanticamente no mesmo controle quando possível;
- playback source/position/session preservados;
- download/recording permanece ativo;
- remote pairing/session permanece ativo;
- somente copy/index/presentation caches são invalidados.


## Declarative registries/options V10

- ContentDisplayMatchMode: storageKey round-trip idêntico em en/pt-BR; labels mudam;
- AndroidVideoRendererMode: storageKey/videoOutput/hardwareDecoder invariantes; label/description localizados;
- external players: brand/executable/bundle/command invariantes; System Default/Custom App/Custom Command/descriptions localizados;
- MetadataCategory/PlaylistViewMode: identidade/behavior invariantes; display localizado;
- Settings Search usa os mesmos localized mappers das telas;
- locale flip atualiza option labels sem regravar preferência;
- launch ident brand name só muda quando classificado PRODUCT_COPY; pitch/description app-owned segue locale.

## Temporal/presentation V10

- CalendarTimeFormat labels en/pt-BR;
- device mode respeita preferência 12/24 do dispositivo;
- explicit 12h e 24h não dependem de concatenação literal AM/PM;
- StremioTvNowPlaying não retorna frase final; UI produz Ended/Ends at no locale efetivo;
- Debrify TV StatsTile usa civil-time formatter e copy localizada;
- TV time picker localiza title/instrução/actions mantendo navegação D-pad.

## Compact metrics V10

- YouTube views, Reddit score e Lemmy score com valor canônico intacto;
- bordas 999/1k/1.25k/999999/1M;
- en/pt-BR;
- nenhum `views` app-owned fica embutido em model/service;
- sort/filter/API continuam usando número, não string compactada.

## Coverage closure V10

- todo path do `RuntimeSurfaceUniverse` aparece exatamente uma vez como reachable/classified ou excluded-with-evidence;
- path novo na tree quebra Gate Q até classificação;
- blob SHA alterado em path sensível exige re-scan;
- generated registra generator;
- vendored patch muda ownership quando passa a conter app-owned copy.

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
- Player: **Reprodutor** na copy PT-BR do produto; `player` permanece token/nome técnico em código, protocolo e diagnóstico;
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

1. PR A — dependency preflight (intl 0.20.2) + foundation + tests + ShippingLocales en-only + LocaleFallbackPolicy + localeEpoch + inventory contracts + runtime/remote copy ownership + rich-text/directionality guardrails.
2. PR B — SettingsRows/Settings Search + navigation/profiles/onboarding + SupportRemoteConfig fixed-copy split + language autonyms.
3. PR C — discover/search/details/collections.
4. PR D — addons/sources/filters/debrid/downloads + official engine-catalog localization + plugin download notifications.
5. PR E — Flutter player + TV surfaces.
6. PR F — Android native resources + NativeLocaleBridge + NativeLocaleStore + background notifications.
7. PR G — IPTV/Debrify TV/Stremio TV/tracking/sync/recovery.
8. PR H — Apple/Desktop/Web native surfaces + macOS menu + Windows installer/metadata + Web lang/dir + temporal taxonomy + Unicode search/collation + mixed-language a11y hardening.
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
- OS-owned Apple/installer surfaces não são tratadas como falha do override interno;
- APP_SUPPLIED_SYSTEM_UI cai para en/system de forma segura se o locale efetivo não puder ser materializado;
- falha de LocalizedCopyResolver nunca bloqueia startup/background task.

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

## Risco: build gera copy depois do scanner

Mitigação:

- classificar `BUILD_GENERATED_PRODUCT_COPY`;
- scan de workflow/scripts/templates;
- source-of-truth único para `.desktop`;
- Gate N inspeciona o artifact final.

## Risco: APP_SUPPLIED_SYSTEM_UI fica misturada com chrome do SO

Mitigação:

- classificar por ownership do texto;
- localizar apenas title/label fornecido pelo app;
- documentar que botões/chrome OS-owned podem continuar no idioma do sistema;
- testes não prometem controle que a API não oferece.

## Risco: suite de testes fica presa ao inglês

Mitigação:

- `localizedTestApp` comum;
- delegates completos;
- behavior tests desacoplados de wording;
- localization tests explícitos por locale;
- matriz seletiva en/pt-BR/pseudo em hotspots em vez de quadruplicar toda a suite.

## Risco: scan “100%” ignora package vendorizado

Mitigação:

- inventory por reachability;
- packages runtime em escopo;
- generated/test/dev classificados;
- stale allowlist falha CI.

---

## Risco: language code vira label localizada

**Falha:** matching/persistência quebra quando English vira Inglês.  
**Mitigação:** code canônico permanece identidade; display name é resolução tardia.

## Risco: pt-BR exibe números mas rejeita vírgula ao digitar

**Falha:** interface parece traduzida, porém formulários só aceitam sintaxe inglesa.  
**Mitigação:** LocalizedInputParser + Gate F + testes de input humano, sem tocar tokens técnicos.

## Risco: dispositivo remoto herda a língua do emissor

**Falha:** telefone pt-BR envia mensagem pronta para TV en, produzindo UI misturada.  
**Mitigação:** result/reason code + args no protocolo; localização somente no receptor.

## Risco: exception English cru reaparece como UX

**Falha:** nova integração mostra `e.toString()` diretamente.  
**Mitigação:** UiFailureReason/Resolver, scanner de sinks e contexto localizado para detalhes externos.

## Risco: CI verde mas PiP/Top Shelf/TV keyboard falha em runtime real

**Falha:** artifact contém resources corretos, mas a integração de plataforma usa outro contexto/locale.  
**Mitigação:** Gate O com hardware/OS representativo e evidência por release.


## Risco: listas compostas parecem traduzidas mas mantêm gramática inglesa

Mitigação:

- inventory de joins que alcançam presentation sinks;
- classificação natural/visual/multiline/technical;
- formatter/mensagem locale-aware para listas naturais;
- testes 0/1/2/3+ itens.

## Risco: truncamento/capitalização quebra Unicode válido

Mitigação:

- Gate P;
- grapheme clusters para operações visuais;
- ban de helpers code-unit-based em paths user-facing salvo allowlist técnica;
- casos de teste com combining marks, emoji ZWJ, flags e non-BMP.

## Risco: copy humana escapa por clipboard/export/plugin sem passar por widget

Mitigação:

- outbound text sinks entram no inventário estático;
- ownership explícito do payload;
- technical/user data permanecem invariantes; product copy usa locale efetivo.

## Risco: upstream muda depois da auditoria e o plano continua “verde”

Mitigação:

- `AUDIT_BASELINE_MANIFEST.json` machine-readable;
- Gate Q antes de rebase/promoção;
- diff classificado por reachability;
- full scan na promoção.

## Risco: tag BCP-47 com extension/variant é truncada e vira outra preferência

Mitigação:

- `ProductLocaleId` limitado a language/script/region;
- rejeitar perda silenciosa;
- extension preferences continuam em autoridades próprias;
- testes com script, region, variant e Unicode extension.

## Risco: idioma troca mas cache mantém copy anterior

Mitigação:

- presentation cache keyed por locale ou eliminado;
- estado semântico em vez de frase localizada;
- testes en → pt-BR → en sem restart;
- native/remote/headless incluídos.

## Risco: guia oficial aberto no browser/QR usa idioma diferente do app

Mitigação:

- locale explícito em link quando o destino suporta;
- não confundir browser Accept-Language com App language;
- QR neutro com locale picker/fallback documentado;
- inventário de todo OFFICIAL_PRODUCT_CONTENT acionado pelo app.

## Risco: resposta assíncrona velha sobrescreve locale novo

Mitigação:

- `localeEpoch` monotônico;
- snapshot locale+epoch por operation;
- commit condicional de presentation state;
- cache semântico separado de copy;
- testes de completion fora de ordem.

## Risco: “localizar data” altera regra funcional de provider

Mitigação:

- taxonomia temporal;
- provider calendar/API/path continuam invariantes;
- first-day-of-week é presentation concern;
- testes Trakt/IPTV separando fetch semantics de UI.

## Risco: Android ganha duas autoridades de App language

Mitigação:

- primeiro release sem per-app language do SO;
- Gate H bloqueia `localeConfig`/LocaleManager/AppCompat sem migração;
- `ui_locale_v1` permanece canônico.

## Risco: wrapper Semantics global pronuncia dados externos no idioma errado

Mitigação:

- language attribution scoped por ownership;
- nodes mistos separados quando necessário;
- external/user data sem idioma confiável não recebe tag inventada.

## Risco: “ordenação localizada” usa search-fold e produz ordem incorreta

Mitigação:

- ordem semântica/provider por padrão;
- collator real somente quando requisito;
- Gate F proíbe chamar compareTo/lowercase de collation locale-aware.

## Risco: locale flip reseta estado funcional

Mitigação:

- invariant presentation-only;
- native recreation com state handoff;
- Gate J/O exercitam navigation, focus, player, download e form state.


## Risco: copy em enum/model passa pelo scanner sem chegar como literal ao widget

**Mitigação:** Gate D faz source-to-sink classification de fields/getters/registries e o hotspot manifest V10 fixa exemplos reais.

## Risco: formatação compacta parece “técnica” e mantém decimal/gramática inglesa

**Mitigação:** valores permanecem numéricos; compact notation e substantivos são resolvidos na presentation layer com locale explícito.

## Risco: relatório diz 100% mas existem paths nunca classificados

**Mitigação:** Gate Q valida fechamento por conjuntos contra a Git tree exata e falha em qualquer path runtime relevante sem decisão.

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
- [ ] Todos os roots usam delegates de AppLocalizations + Material + Widgets + Cupertino.
- [ ] showDatePicker/showTimePicker e demais widgets framework-owned exibem copy do locale efetivo.
- [ ] SettingsRows não guarda copy inglesa como identidade.
- [ ] Settings Search é localizada e accent-friendly.
- [ ] Zero lógica depende de display strings, inclusive Download/Recording services nativos.
- [ ] Zero hardcoded UI conhecido fora de allowlist justificada.
- [ ] SupportRemoteConfig não injeta fixed product copy inglesa fora do ARB.
- [ ] Campaign/remote product copy possui schema locale-aware e fallback testado.
- [ ] Catálogo oficial de engines/onboarding possui política locale-aware para copy editorial.
- [ ] Runtime-loaded assets user-facing estão no inventory/completeness report.
- [ ] Notifications via background_downloader estão localizadas onde user-facing.
- [ ] LocalizedCopyResolver/equivalente fornece copy fora de BuildContext sem segunda autoridade de locale.
- [ ] APP_SUPPLIED_SYSTEM_UI está inventariada e localizada onde o app controla a copy.
- [ ] FilePicker dialogTitle e PiP RemoteAction não possuem English-only app copy em shipping pt-BR.
- [ ] Text.rich/RichText/TextSpan/TextPainter/CustomPainter user-facing estão localizados ou classificados.
- [ ] Hardcoded directionality app-owned possui zero findings não classificados.
- [ ] Language picker usa autônimos estáveis.
- [ ] MarkdownBody/Markdown user-facing possui source ownership/classification.
- [ ] Release notes oficiais possuem política editorial de locale registrada.
- [ ] WebDAV Setup guide/link/QR possui destino com política de locale.
- [ ] Runtime visual assets com potencial texto foram auditados/classificados.
- [ ] Formatters são locale-aware onde aplicável.
- [ ] Android native values-pt-rBR completo.
- [ ] Zero android:text/contentDescription/hint e demais XML string-bearing user-facing fora de allowlist.
- [ ] Android resource parity inclui tipo, plurals, arrays e placeholder/format parity.
- [ ] PiP RemoteAction Play/Pause/Next resolve via resources e foi testado.
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
- [ ] Linux `.desktop` possui uma única fonte canônica locale-aware.
- [ ] AppImages x86_64 e arm64 contêm `Comment[pt_BR]` no `.desktop` final.
- [ ] `.github/workflows`/packaging scripts não reintroduzem user-facing English-only copy.
- [ ] Windows installer possui PT-BR e custom copy localizada.
- [ ] Windows Runner.rc não contém metadata template com.example.
- [ ] Web manifest/index sem placeholder Flutter.
- [ ] Web documentElement lang/dir acompanha locale efetivo.
- [ ] Semantics/contentDescription localizados.
- [ ] Semantics locale attribution foi testada com App language != system locale; wrapper `localeForSubtree` aplicado se necessário no Flutter 3.44.8.
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
- [ ] test harness localizado comum cobre widgets migrados; asserts não essenciais não dependem de wording inglês.
- [ ] Gate N confirma localização dentro dos artifacts finais publicados.
- [ ] Android native tests passam.
- [ ] build matrix dos artifacts publicados passa.
- [ ] revisão humana PT-BR em contexto concluída.
- [ ] README/documentação atualizados.
- [ ] rollback path testado.

- [ ] Language display names próprios usam code/semantic identity e resolver localizado; zero mapa inglês duplicado user-facing fora de allowlist.
- [ ] Dart/Android native possuem paridade para nomes de idioma gerados pelo produto.
- [ ] Entradas numéricas humanas são parseadas pelo locale efetivo e persistidas em forma canônica.
- [ ] Inputs técnicos (IP/URL/porta/PIN/IDs/schema) continuam invariantes.
- [ ] TvTextField/tv_keyboard não possuem submit/action/semantics English-only em shipping pt-BR.
- [ ] Platform/native/service errors usam reason codes estáveis; raw exception text não é contrato de UI.
- [ ] Remote cross-device localiza resultados no dispositivo receptor, não no emissor.
- [ ] Compatibilidade com payload remoto legado foi testada durante a migração.
- [ ] Gate O real-device runtime smoke passa com evidência por artifact/locale/device.
- [ ] Accessibility smoke cobre pelo menos um cenário system language != App language.
- [ ] ARB/allowlist stale-orphan gate está verde.

- [ ] Toda composição user-facing por `.join`/separador possui classificação natural/visual/multiline/technical.
- [ ] Listas naturais possuem formatter/mensagem locale-aware e testes 0/1/2/3+ em en/pt-BR.
- [ ] Zero truncamento/capitalização visual conhecido baseado em code unit fora de allowlist técnica justificada.
- [ ] Truncamento/iniciais/editing user-facing preservam grapheme clusters (combining marks, non-BMP, ZWJ, flags).
- [ ] Casing pós-localização é zero salvo exceção explicitamente classificada.
- [ ] Clipboard/share/export/report/plugin human-text sinks possuem ownership e locale policy.
- [ ] Gate P — Unicode/composition/outbound sinks está verde.
- [ ] Gate Q — upstream baseline drift/audit freshness está verde.
- [ ] Gate P está definido no plano canônico, não apenas em documento auxiliar.
- [ ] Todos os documentos normativos usam o registro canônico de IDs `0, A..Q`, sem Gate 1..14/aliases.
- [ ] `docs/AUDIT_BASELINE_MANIFEST.json` corresponde ao commit/tree realmente auditado e a árvore recursive foi completa.
- [ ] `ProductLocaleId` limita a preferência de UI ao subconjunto language/script/region sem perda silenciosa de variants/extensions/private-use.
- [ ] Android baseline lê exatamente `FlutterSharedPreferences / flutter.ui_locale_v1` e o contract test Dart→native passa.
- [ ] Troca en → pt-BR → en sem restart não deixa presentation cache stale em superfícies long-lived.
- [ ] OFFICIAL_PRODUCT_CONTENT aberto em browser/QR possui política que distingue App language, browser/system language e outro dispositivo.

- [ ] Calendar/date presentation não contém tabelas manuais English-only em paths shipping.
- [ ] `HUMAN_DATE` e `CIVIL_TIME` usam locale efetivo; timecode/protocolo/filename/diagnóstico permanecem invariantes por classificação.
- [ ] Primeiro dia visual da semana não altera regras funcionais de calendário/API/provider.
- [ ] Models/services não devolvem presentation date/size/relative-time strings como contrato quando podem devolver dados tipados.
- [ ] `localeEpoch` protege todo async presentation commit relevante contra completion fora de ordem.
- [ ] `LocaleFallbackPolicy` é explícita/versionada e independente de `ShippingLocales`.
- [ ] Settings Search passa corpus Unicode NFC/NFD/combining marks PT-BR.
- [ ] Nenhuma ordenação é declarada locale-aware usando apenas `String.compareTo`/lowercase/search-fold.
- [ ] Android pt-BR v1 não cria segunda autoridade via per-app language do SO.
- [ ] Semantics language attribution é scoped por ownership e testada com conteúdo externo em outro idioma.
- [ ] Troca de App language preserva navigation, playback, downloads/recordings, pairing e input não salvo.
- [ ] Runtime roots sob `packages/**` alcançáveis foram classificados e incluídos em D/H/N/Q.

---


- [ ] App-owned option/registry copy não permanece congelada em enum/model/service getter quando alcança UI.
- [ ] `CalendarTimeFormat`, Stremio TV now-playing, Debrify TV stats e TV time picker passam os testes V10.
- [ ] Compact metrics user-facing não usam `toStringAsFixed + K/M/B/views` como formatter final fora da presentation layer.
- [ ] O coverage ledger fecha exatamente sobre o `RuntimeSurfaceUniverse` da tree auditada, sem path órfão.

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

A V10 acrescenta uma condição adicional à alegação de completude: além de zero finding conhecido, o universo de paths runtime/product da Git tree precisa estar fechado por classificação machine-readable. Scanner sem coverage closure não prova 100%.

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

A V10 preserva e reforça a condição temporal introduzida na V8/V9: cobertura só é atribuída ao SHA/tree registrado no manifest. Se a base upstream muda, a afirmação de completude expira até Gate Q revalidar o delta.

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
- `AUDIT_BASELINE_MANIFEST.json` + resultado Gate Q para a base usada;
- registro canônico dos gates executados por ID, sem aliases;
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
- limitações OS-owned registradas;
- relatório Gate N/inspeção do artifact final quando o PR toca packaging/native resources;
- prova de que build-generated copy não sobrescreveu a fonte localizada.

A meta da V10 é transformar “100%” de uma promessa subjetiva em um conjunto auditável de provas reproduzíveis que cobre source code, plataformas nativas, assets de runtime, copy oficial remota, app-supplied system UI, build-generated product copy, testes e o conteúdo efetivamente presente nos artifacts publicados.
