# Reauditoria V2 — Debrify i18n

**Data:** 2026-09-25  
**Upstream:** varunsalian/debrify  
**Release verificada:** v0.10.0-beta.1  
**Commit verificado:** 9619c10b06ee919cacbe996b30be7739dc09c6d6  
**Plano resultante:** PLANO_MESTRE_V2.md

## Conclusão

A direção técnica do V1 era correta, mas o arquivo estava truncado no início da Fase 10 e faltavam contratos explícitos para várias superfícies importantes.

A baseline upstream foi reconfirmada no mesmo dia da auditoria e continua atual.

## Evidências reconfirmadas

- workflow usa Flutter 3.44.8;
- pubspec possui intl ^0.19.0;
- pubspec possui shared_preferences ^2.2.2;
- não há flutter_localizations;
- não há AppLocalizations;
- não há supportedLocales;
- lib/utils/formatters.dart usa padrões de data fixos e decimal manual;
- Android já possui strings.xml parcial;
- existem strings hardcoded em Dart e código nativo;
- o projeto possui múltiplas superfícies de bootstrap/recovery.

## Novos achados críticos da V2

### 1. V1 truncado

PLANO_MESTRE_V1.md termina na abertura de Fase 10 / Android. O restante do plano não estava presente no arquivo.

### 2. SettingsRows precisa de refactor arquitetural

lib/screens/settings/widgets/settings_widgets.dart mantém copy inglesa em SettingsRowContent e SettingsRows por meio de static const.

Consequência:

não basta trocar widgets isolados por context.l10n. A identidade da row e a copy precisam ser separadas.

### 3. Settings Search é English-centric

lib/screens/settings/settings_search.dart:

- indexa title/subtitle/category/keywords;
- converte com toLowerCase;
- injeta literalmente a palavra settings no haystack;
- possui Search Settings, Search settings…, Clear e empty state hardcoded.

O índice precisa ser reconstruído no locale atual e possuir política explícita de normalização/diacríticos.

### 4. Display string participa da lógica

Foi confirmado em lib/screens/video_player/widgets/source_sheet.dart um grupo com id all e label All sources, além de comparação do próprio label.

Isso vira um requisito explícito:

display strings nunca podem ser identidade, sentinel, branch condition ou valor persistido.

### 5. User-facing text fora de widgets

Há helpers/utils/services que retornam sentenças visíveis. Exemplo: lib/utils/file_utils.dart produz mensagem inglesa de compatibilidade de formato.

O scanner precisa alcançar camada não visual.

### 6. Case transforms exigem classificação

Há toUpperCase em caminhos técnicos e em apresentação.

A auditoria não deve proibir todo uppercase. Ela deve distinguir:

- parsing/protocolo;
- storage;
- token técnico;
- copy;
- dados externos;
- input/teclado.

### 7. Mais de um player Android nativo

O manifest e o código confirmam AndroidTvTorrentPlayerActivity e TorboxTvPlayerActivity.

Ambos entram na matriz locale system/override e resource tests.

### 8. Apple possui copy de sistema não coberta por ARB

iOS e macOS possuem NSLocalNetworkUsageDescription em inglês no Info.plist.

Essas mensagens precisam de localização nativa via recursos de plataforma.

tvOS também possui Runner e Top Shelf extension que entram no inventário.

### 9. Web shell possui placeholder

web/manifest.json e web/index.html ainda contêm A new Flutter project.

Isso não é apenas tradução; é uma pendência de qualidade de shell/PWA que deve ser corrigida junto da fase web.

### 10. Shipping locale precisa ser explícito

gen_l10n pode gerar suporte para todo ARB presente. Durante migração, isso não deve transformar automaticamente PT-BR parcial em locale público.

O V2 define ShippingLocales separado do catálogo gerado.

## Riscos prioritários

1. mixed-language release;
2. display text usado como controle;
3. divergência Flutter x native player;
4. Settings centralizado em copy const;
5. strings fora do scanner;
6. TV focus/overflow;
7. permission descriptions não localizadas;
8. locale entrando em sync/profile;
9. futuras linguagens bloqueadas por input/font/RTL.

## Decisão

PLANO_MESTRE_V2.md passa a ser a especificação atual.

PLANO_MESTRE_V1.md permanece apenas para histórico/audit trail.
