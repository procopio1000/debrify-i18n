# Auditoria V9 — Debrify i18n/l10n

**Data:** 2026-09-25  
**Plano resultante:** `PLANO_MESTRE_V9.md`  
**Upstream:** `varunsalian/debrify`  
**Release revalidada:** `v0.10.0-beta.1`  
**Commit revalidado:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`  
**Tree revalidada:** `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`

## Estado do baseline

A comparação `9619c10... -> main` do upstream está **identical** no momento desta auditoria: 0 commits à frente, 0 atrás. A recursive tree auditada continua completa. A V9, portanto, não mascara drift: ela aprofunda a especificação sobre a mesma árvore V8.

## Achados novos

### 1. Calendar presentation hardcoded

Foram confirmadas tabelas e frases de calendário em inglês em runtime first-party:

- `lib/widgets/trakt_calendar_day_sheet.dart`;
- `lib/screens/trakt_calendar_screen.dart`;
- `lib/widgets/iptv/iptv_epg_panel.dart`;
- `lib/screens/settings/recordings_page.dart`;
- `lib/widgets/iptv/spotlight/spotlight_live_timeline.dart`;
- `lib/services/trakt/trakt_episode_model.dart`.

A V8 cobria formatters genericamente, mas não tornava esses hotspots nem a detecção de month/weekday tables uma prova explícita.

**Correção V9:** hotspot manifest + Gate F + matriz temporal.

### 2. Presentation formatting dentro de models/services

`lib/models/rd_user.dart` expõe `formattedExpiration`, consumido pela UI; outros models fazem `_formatDate`, `N/A` e/ou size formatting.

**Risco:** locale entra no domínio ou o domínio congela formato inglês/técnico como presentation.

**Correção V9:** domain expõe tipos canônicos; presentation formata.

### 3. Taxonomia temporal ausente

Há clocks user-facing, clocks deliberadamente 24h, media timecodes, filenames, APIs e logs. Tratar todos como “formatar pelo locale” seria regressão.

`TraktCalendarService`, por exemplo, alinha chunk à segunda-feira por contrato funcional; isso não deve mudar porque a UI usa outro primeiro dia visual da semana.

**Correção V9:** HUMAN_DATE, CIVIL_TIME, PRODUCT_FIXED_CLOCK, MEDIA_TIMECODE, PROTOCOL_DATE_TIME, FILENAME_TIMESTAMP, DIAGNOSTIC_TIMESTAMP e PROVIDER_CALENDAR_RULE.

### 4. Race assíncrona de locale

Presentation cache keyed por locale não impede por si só:

1. request en começa;
2. usuário muda pt-BR;
3. request pt-BR termina;
4. request en termina atrasado e publica copy velha.

**Correção V9:** `localeEpoch`/snapshot e commit condicional de presentation state.

### 5. Android per-app language sem decisão final

Busca na baseline não encontrou `android:localeConfig`, `LocaleManager`, `AppCompatDelegate.setApplicationLocales` ou `generateLocaleConfig`.

**Correção V9:** primeiro release pt-BR mantém `ui_locale_v1/AppLocaleController` como autoridade única. Qualquer integração futura com seletor do SO requer migração dedicada.

### 6. Fallback policy derivada do conjunto shipping

A regra `pt -> pt-BR` não deve existir apenas porque hoje só há um português.

**Correção V9:** `LocaleFallbackPolicy` versionada e separada de `ShippingLocales`.

### 7. Unicode search conformance

A V8 exige busca accent-friendly, mas não fechava equivalência composta/decomposta.

**Correção V9:** corpus NFC/NFD/combining marks; zero map ad hoc por widget.

### 8. Collation

Search folding não é collation. `String.compareTo`/lowercase não devem ser vendidos como ordenação locale-aware.

**Correção V9:** ordem estável por padrão; collator real somente quando requisito.

### 9. Semantics mixed-language

Um wrapper global de locale pode marcar user/external data como se estivesse no App language.

**Correção V9:** language attribution scoped por ownership, com nested semantics quando houver metadata confiável.

### 10. Continuidade funcional no locale flip

Trocar idioma deve ser presentation-only.

**Correção V9:** testes de navigation, focus, playback, downloads/recordings, pairing e form state.

### 11. Editorial contract ainda tinha decisão aberta

O glossário dizia que `Player` poderia virar “Reprodutor”, ao mesmo tempo em que o plano exigia consistência global.

**Correção V9:** a copy PT-BR usa **Reprodutor**; `player` continua permitido como token técnico/código/protocolo. O glossário também passa a fixar termos temporais básicos usados pelos novos hotspots.

### 12. Packages runtime

A árvore inclui forks/packages com código Dart e nativo. “packages/” não pode ser uma fronteira implícita de exclusão.

**Correção V9:** roots runtime recursivos e classificação FIRST_PARTY_FORK / VENDORED_THIRD_PARTY / GENERATED nos Gates D/H/N/Q.

## Resultado

A V9 não adiciona outro gate e não renumera nada. O registro continua:

`0, A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q`.

Isso preserva automação já desenhada e move os novos achados para os gates semanticamente corretos.
