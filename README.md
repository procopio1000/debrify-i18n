# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Estado atual

**Plano canônico:** PLANO_MESTRE_V3.md

A V3 foi re-auditada contra o upstream e contra o toolchain Flutter 3.44.8 em 2026-09-25. Ela fecha blockers de dependência, runtime nativo em background, ownership de locale por plataforma, Windows/macOS/Web e scan por reachability que ainda estavam subespecificados na V2.

V1 e V2 permanecem no repositório apenas como histórico/audit trail.

## Alvo verificado

- Upstream: varunsalian/debrify
- Baseline: v0.10.0-beta.1
- Commit auditado: 9619c10b06ee919cacbe996b30be7739dc09c6d6
- Primeiro locale completo: pt-BR
- Template canônico: en

## Arquivos

- PLANO_MESTRE_V3.md — plano canônico atual, toolchain-verified e implementation-ready
- PLANO_MESTRE_V2.md — histórico/superseded pela V3
- PLANO_MESTRE_V1.md — histórico/superseded
- docs/AUDITORIA_V3.md — terceira auditoria e evidências bloqueadoras
- docs/AUDITORIA_V2.md — histórico da segunda auditoria
- docs/AUDITORIA_BASELINE.md — evidências da auditoria inicial
- docs/ARQUITETURA.md — arquitetura V3
- docs/GLOSSARIO_PT_BR.md — terminologia inicial
- docs/MATRIZ_TESTES.md — matriz de testes V3
- docs/CI_QUALITY_GATES.md — quality gates V3

## O que a V3 acrescenta

Além da infraestrutura já consolidada na V2, a V3 cobre explicitamente:

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

## Regra central

A linguagem da interface é independente de:

- linguagem de metadados;
- linguagem de áudio;
- linguagem de legendas;
- região.

Texto localizado nunca pode funcionar como ID, valor persistido ou condição de lógica.
