# Auditoria V6 — Debrify i18n/l10n

**Data:** 2026-09-25  
**Plano resultante:** `PLANO_MESTRE_V6.md`  
**Upstream:** `varunsalian/debrify`  
**Release revalidada:** `v0.10.0-beta.1`  
**Commit revalidado:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`  
**Git tree de referência:** `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`

## Revalidação do baseline

Na auditoria V6, a release mais recente do upstream continuava `v0.10.0-beta.1` e `main` continuava exatamente no commit auditado pela V5. Portanto, os novos achados abaixo são lacunas reais da mesma baseline, não drift entre versões.

## Achados novos

### 1. Nomes de idiomas duplicados em inglês

Evidências:

- `lib/models/stremio_subtitle.dart`: `_formatLanguageCode` mantém mapa `eng -> English`, `spa -> Spanish`, `por -> Portuguese`, `por-br -> Portuguese (Brazil)`;
- `android/app/src/main/kotlin/com/debrify/app/subtitle/StremioSubtitleService.kt`: outro `languageNames` equivalente;
- `lib/screens/settings/filter_settings_page.dart`: opções `English`, `Hindi`, `Spanish`, ... `Multi-Audio`;
- `lib/screens/settings_screen.dart`: aliases/labels ingleses de áudio/legenda.

**Falha potencial:** ARB pode chegar a 100%, mas listas de idioma continuarem inglesas.

**Correção V6:** code/semantic identity separada de display name localizado; resolver comum e paridade Dart/native.

### 2. Entrada decimal humana ainda é English-only

`lib/screens/collections/collection_editor_screen.dart` valida rating com `double.tryParse(value)` e `double.parse(value)`.

**Falha potencial:** em pt-BR, `8,5` é entrada natural e falha apesar de a UI estar traduzida.

**Correção V6:** `LocalizedInputParser` conceitual, locale explícito e separação rígida de tokens técnicos (IP/URL/porta/PIN/IDs/schema).

### 3. Teclado Debrify TV possui copy própria fora do inventário explícito

`lib/widgets/tv_text_field.dart` cria defaults `Search`, `Go`, `Next`, `Send`, `Done`. Diversos callers passam `keyboardSubmitLabel` em inglês. `lib/widgets/tv_keyboard.dart` inclui `Clear` e outras ações/semantics.

**Correção V6:** esses parâmetros/actions tornam-se sinks explícitos no Gate D e na matriz de testes.

### 4. Mensagens humanas atravessam Dart/native

Exemplos:

- `LocalSourceAccess.kt`: `result.error("busy", "A file picker is already open.", null)`;
- `picker_unavailable` com mensagem inglesa;
- outros boundaries podem repassar `e.message`, `PlatformException.message` ou exception string.

**Falha potencial:** a UI exibe texto inglês cru ou passa a depender dele para comportamento.

**Correção V6:** machine reason code + structured args + diagnostic detail; apresentação localizada no destino.

### 5. Remote transporta mensagem humana entre dispositivos

O subsistema Remote mantém resultados `({bool ok, String message})`, labels e outcomes que podem chegar diretamente a SnackBar/Dialog.

**Falha arquitetural:** App language já é definido como device-local. Logo um telefone em pt-BR não pode decidir a língua da mensagem exibida por uma TV em inglês, nem vice-versa.

**Correção V6:** protocolo canônico usa `resultCode/reasonCode` + args; cada runtime receptor localiza. Payload legado com `message` só existe como compatibilidade transitória.

### 6. Artifact green não prova integração em hardware/OS

PiP, notification channels, FilePicker, permission prompts, Top Shelf, TV input/focus e accessibility podem divergir em runtime mesmo com recursos presentes no artifact.

**Correção V6:** novo **Gate O — Real-device runtime localization smoke** com artifact SHA, device/OS, system locale, App language e evidência.

### 7. Higiene do catálogo

Key parity não detecta tradução morta. A V6 adiciona detecção de ARB keys órfãs e allowlists stale para manter o catálogo sustentável depois da migração.

## Resultado

A V5 continua historicamente válida, mas a V6 é mais completa porque fecha três classes que source scanners tradicionais quase sempre perdem:

1. **semantic display data** (language names);
2. **localized input**, não apenas localized output;
3. **locale crossing runtime/device boundaries**.

A V6 também adiciona evidência de runtime real para complementar os Gates A–N.

## Critério de 100%

“100%” não é tratado como alegação impossível de provar por inspeção estática. Na V6 ele significa que toda copy própria alcançável possui:

- owner/classificação;
- semantic identity separada da apresentação;
- locale authority definida;
- parser/formatter correto quando há entrada/saída;
- reason/result contract nas fronteiras;
- teste automatizado;
- inspeção no artifact;
- e, quando a plataforma exige, prova em runtime real.
