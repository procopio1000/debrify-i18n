# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Estado atual

**Plano canônico:** PLANO_MESTRE_V2.md

A V2 foi re-auditada contra o upstream em 2026-09-25 e corrige lacunas do V1, incluindo o fato de que PLANO_MESTRE_V1.md estava truncado no início da Fase 10.

O V1 permanece no repositório apenas como histórico.

## Alvo verificado

- Upstream: varunsalian/debrify
- Baseline: v0.10.0-beta.1
- Commit auditado: 9619c10b06ee919cacbe996b30be7739dc09c6d6
- Primeiro locale completo: pt-BR
- Template canônico: en

## Arquivos

- PLANO_MESTRE_V2.md — plano atual, completo e implementation-ready
- PLANO_MESTRE_V1.md — histórico/superseded
- docs/AUDITORIA_V2.md — reauditoria e achados adicionais
- docs/AUDITORIA_BASELINE.md — evidências da auditoria inicial
- docs/ARQUITETURA.md — arquitetura V2
- docs/GLOSSARIO_PT_BR.md — terminologia inicial
- docs/MATRIZ_TESTES.md — matriz de testes V2
- docs/CI_QUALITY_GATES.md — quality gates V2

## O que a V2 acrescenta

Além da infraestrutura Flutter/ARB original, a V2 cobre explicitamente:

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

## Regra central

A linguagem da interface é independente de:

- linguagem de metadados;
- linguagem de áudio;
- linguagem de legendas;
- região.

Texto localizado nunca pode funcionar como ID, valor persistido ou condição de lógica.
