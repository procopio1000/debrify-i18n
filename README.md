# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Estado atual

**Plano canônico:** PLANO_MESTRE_V7.md

A V7 foi re-auditada contra a mesma baseline exata do upstream em 2026-09-25 e preserva todos os hardenings da V6. Ela fecha também composição de listas user-facing, truncamento/capitalização Unicode por grapheme cluster, casing pós-localização e sinks de texto humano fora da árvore de widgets (clipboard/share/export/report/plugin). O Gate P torna essas classes verificáveis em CI.

V1–V6 permanecem no repositório como histórico/audit trail.

## Alvo verificado

- Upstream: varunsalian/debrify
- Baseline: v0.10.0-beta.1
- Commit auditado: 9619c10b06ee919cacbe996b30be7739dc09c6d6
- Primeiro locale completo: pt-BR
- Template canônico: en

## Arquivos

- PLANO_MESTRE_V7.md — plano canônico atual, toolchain/cross-runtime/cross-device/packaging/runtime-verified-by-design, unicode/composition-hardened e implementation-ready
- PLANO_MESTRE_V6.md — histórico/superseded pela V7
- PLANO_MESTRE_V5.md — histórico/superseded pela V6
- PLANO_MESTRE_V4.md — histórico/superseded pela V5
- PLANO_MESTRE_V3.md — histórico/superseded pela V4
- PLANO_MESTRE_V2.md — histórico/superseded pela V3
- PLANO_MESTRE_V1.md — histórico/superseded
- docs/AUDITORIA_V6.md — sexta auditoria, semantic boundaries/cross-device/input/runtime smoke e evidências
- docs/AUDITORIA_V5.md — histórico da quinta auditoria
- docs/AUDITORIA_V4.md — histórico da quarta auditoria
- docs/AUDITORIA_V3.md — histórico da terceira auditoria
- docs/AUDITORIA_V2.md — histórico da segunda auditoria
- docs/AUDITORIA_BASELINE.md — evidências da auditoria inicial
- docs/ARQUITETURA.md — arquitetura V6
- docs/GLOSSARIO_PT_BR.md — terminologia inicial
- docs/MATRIZ_TESTES.md — matriz de testes V6
- docs/CI_QUALITY_GATES.md — quality gates V6

## O que a V6 consolida e acrescenta

Além de todos os contratos V1–V5, a V6 acrescenta explicitamente:

- nomes de idiomas como apresentação localizada, mantendo ISO/BCP-47 como identidade;
- parsing locale-aware de entrada numérica humana sem alterar IP/URL/PIN/IDs/protocolos;
- `keyboardSubmitLabel` e action/semantics do teclado Debrify TV como sinks explícitos;
- reason/result codes estáveis em boundaries Dart/native/services;
- Remote multi-device localizando no receptor, não no emissor;
- Gate O de runtime real para PiP/notifications/FilePicker/Top Shelf/TV/accessibility;
- detecção de ARB/allowlist órfãos/stale;

A V6 também preserva integralmente o hardening acumulado anteriormente:

- ShippingLocales separado dos locales gerados;
- SettingsRows e Settings Search localizados por identidade estável;
- proibição de usar display text como lógica/persistência;
- scanner de strings fora de widgets;
- ambos os players Android nativos;
- NativeLocaleBridge;
- iOS/tvOS/macOS e InfoPlist.strings;
- tvOS Top Shelf;
- Windows/Linux;
- Web/PWA e remoção de placeholders;
- case transforms;
- RTL;
- fontes;
- acessibilidade;
- teclado/input TV;
- rollout, rollback e PR strategy;
- Definition of Done mensurável.
- compatibilidade real `flutter_localizations` / `intl 0.20.2` no Flutter 3.44.8;
- remoção de `synthetic-package` obsoleto;
- NativeLocaleStore para Services/Receivers Android sem Flutter ativo;
- remoção de lógica nativa baseada em frases inglesas;
- scan de XML Android e packages runtime por reachability;
- ownership matrix para superfícies controladas pelo SO;
- macOS MainMenu;
- Windows installer PT-BR e limpeza de `com.example`;
- Web `lang`/`dir` em runtime;
- locale de voz/input independente;
- bidi isolation;
- evidência mínima reproduzível por PR.
- `REMOTE_PRODUCT_COPY` para campanhas/suporte oficiais;
- contrato locale-aware para catálogo remoto oficial de engines;
- runtime assets JSON/YAML/Markdown/CSV no grafo de reachability;
- notificações `background_downloader` fora do Android nativo;
- `Text.rich`/`TextSpan`/`TextPainter`/CustomPainter como sinks de UI;
- rich text reorder-safe por placeholders semânticos;
- autônimos estáveis no seletor de idioma;
- Gate L para copy remota/runtime;
- Gate M para directionality e inline-text safety;
- completeness report além de ARB/native resources.
- Markdown/release notes oficiais com ownership explícito;
- WebDAV Setup guide/link/QR com política de locale;
- runtime visual assets com potencial texto;
- separação entre CORE_UI_COMPLETENESS e PRODUCT_EXPERIENCE_COMPLETENESS;
- `APP_SUPPLIED_SYSTEM_UI` para PiP/FilePicker e outras superfícies desenhadas pelo SO com copy fornecida pelo app;
- `BUILD_GENERATED_PRODUCT_COPY` para workflow/scripts que geram metadata user-facing;
- uma única fonte `.desktop` Linux e verificação nos AppImages x86_64/arm64;
- `LocalizedCopyResolver` para services/background sem `BuildContext`;
- delegates Material/Widgets/Cupertino explícitos;
- `localizedTestApp`/test harness para evitar suite presa ao inglês;
- resource placeholder/type parity no Android;
- Gate N para provar localização dentro dos artifacts finais.

## Regra central

A linguagem da interface é independente de:

- linguagem de metadados;
- linguagem de áudio;
- linguagem de legendas;
- região.

Texto localizado nunca pode funcionar como ID, valor persistido ou condição de lógica.
