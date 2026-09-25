# debrify-i18n

Projeto de planejamento e especificação para implementar internacionalização completa no Debrify.

## Alvo

- Upstream: `varunsalian/debrify`
- Baseline: `v0.10.0-beta.1`
- Commit auditado: `9619c10b06ee919cacbe996b30be7739dc09c6d6`
- Primeiro locale completo: `pt-BR`

## Arquivos

- `PLANO_MESTRE_V1.md` — plano ultra abrangente
- `docs/AUDITORIA_BASELINE.md` — evidências do estado atual
- `docs/ARQUITETURA.md` — arquitetura proposta
- `docs/GLOSSARIO_PT_BR.md` — terminologia inicial
- `docs/MATRIZ_TESTES.md` — testes necessários
- `docs/CI_QUALITY_GATES.md` — quality gates

## Objetivo

Não é apenas traduzir strings. É criar uma fundação permanente de i18n/l10n para:

- Flutter;
- Android TV nativo;
- TV/desktop/mobile/web;
- acessibilidade;
- formatação;
- idiomas futuros;
- CI contra regressões.

## Regra central

A linguagem da interface é independente de:

- linguagem de metadados;
- linguagem de áudio;
- linguagem de legendas;
- região.