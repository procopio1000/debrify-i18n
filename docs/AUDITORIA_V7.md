# Auditoria V7 — Debrify i18n/l10n

**Data:** 2026-09-25  
**Plano resultante:** `PLANO_MESTRE_V7.md`  
**Upstream:** `varunsalian/debrify`  
**Release revalidada:** `v0.10.0-beta.1`  
**Commit revalidado:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`

## Revalidação do baseline

O upstream continua exatamente no commit auditado pela V6. Os achados abaixo são lacunas de especificação da mesma baseline, não drift de versão.

## Achados novos

### 1. Composição manual de listas user-facing

Evidências confirmadas:

- `lib/widgets/onboarding/steps/done_step.dart`: `summary.services.join(', ')` e `summary.trackers.join(', ')`;
- `lib/widgets/rewatch_progress_dialog.dart`: lista de falhas inserida em frase via `failures.join(', ')`;
- `lib/utils/continue_watching_presentation.dart`: metadata compacta unida por ` · `;
- `lib/widgets/remote/remote_send_workspace.dart`: labels de falha unidos por newline.

**Falha potencial:** uma aplicação pode ter 100% das keys ARB e ainda montar listas com gramática/separadores fixos de inglês.

**Correção V7:** classificação obrigatória em NATURAL_LANGUAGE_LIST, VISUAL_METADATA_LIST, MULTILINE_STRUCTURED_LIST ou TECHNICAL_LIST; formatter/mensagem locale-aware para listas naturais; testes 0/1/2/3+.

### 2. Truncamento visual por UTF-16 code unit

Evidência confirmada:

- `lib/widgets/pikpak_folder_picker_dialog.dart::_truncateFolderName` usa `name.length` e `substring`.

**Falha potencial:** cortar surrogate pair, combining mark, emoji ZWJ ou flag no meio, produzindo apresentação incorreta mesmo sem haver qualquer literal inglês.

**Correção V7:** operações visuais de count/truncate/initials/edit passam a ser grapheme-safe; Gate P detecta helpers code-unit-based em presentation paths.

### 3. Capitalização/indexação por primeiro code unit

Evidências:

- `lib/screens/addons/addon_hub_screen.dart::_capitalize` usa `s[0]` + `substring(1)`;
- `lib/widgets/addon_identity.dart` já usa `.characters.first`, mas aplica casing depois e portanto ainda requer classificação de apresentação.

**Falha potencial:** assumir que “primeiro caractere” = primeiro code unit e que casing inglês é universal.

**Correção V7:** casing pós-l10n é proibido por padrão; dados externos/user data preservam conteúdo; qualquer transformação necessária precisa ser grapheme-safe e semanticamente classificada.

### 4. Texto humano pode sair por sinks sem widget

A baseline usa Clipboard em vários fluxos e passa copy para APIs de sistema/plugins. Muitos payloads atuais são corretamente técnicos — URL, PIN, token, código — mas o scanner não pode assumir que `ClipboardData(text: ...)`, share/export/report e nomes/títulos sugeridos nunca carregarão product copy.

**Correção V7:** esses canais entram no inventário de presentation/output sinks e recebem ownership explícito.

### 5. Documentos auxiliares ainda anunciavam V4

`docs/ARQUITETURA.md`, `docs/CI_QUALITY_GATES.md` e `docs/MATRIZ_TESTES.md` preservavam cabeçalhos “V4”, com hardenings posteriores anexados.

**Falha operacional:** um agente de implementação pode interpretar V4 como versão normativa ou separar indevidamente apêndices V6 do contrato principal.

**Correção V7:** cabeçalhos atualizados para V7 e hardening novo integrado explicitamente.

## Gate novo

### Gate P — Unicode, composition e outbound text sinks

O gate cobre:

- joins/separadores user-facing sem classificação;
- truncate/capitalize/initials por code unit em presentation paths;
- casing pós-localização;
- clipboard/share/export/report/plugin/system human-text sinks sem ownership;
- persistência indevida de resultado localizado/composicionado.

## Critério de completude

A V7 não substitui os Gates A–O; ela fecha uma classe diferente: **a string pode estar traduzida corretamente e ainda ser composta ou manipulada de forma linguisticamente/Unicode incorreta**.

A promoção de pt-BR continua condicionada aos gates anteriores e agora também ao Gate P.
