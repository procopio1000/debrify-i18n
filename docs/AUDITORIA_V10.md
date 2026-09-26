# Auditoria V10 — Debrify i18n/l10n

**Data:** 2026-09-26  
**Plano resultante:** `PLANO_MESTRE_V10.md`  
**Upstream:** `varunsalian/debrify`  
**Release revalidada:** `v0.10.0-beta.1`  
**Commit revalidado:** `9619c10b06ee919cacbe996b30be7739dc09c6d6`  
**Tree revalidada:** `cffb6c9d272c662eb0f5f93e6376cb4b239a57c3`

## Estado do baseline

A comparação entre o commit auditado e o `main` upstream permanece **identical**: zero commits à frente/atrás no momento desta auditoria. A V10, portanto, aprofunda a mesma árvore da V9; não mistura novos achados com drift de código.

A recursive tree continua completa (`truncated=false`). As contagens de blobs usadas pelo plano permanecem as do manifest de baseline.

## Objetivo específico da V10

A V9 já cobre sinks diretos, models/services, formatters e reachability de forma ampla. A V10 procurou falhas que sobreviveriam a um scanner literal/sink-oriented:

1. copy app-owned armazenada em enum/model/extension/registry e consumida indiretamente;
2. presentation strings finais produzidas em models/services;
3. relógios/compact numbers manuais que não estavam no hotspot manifest;
4. ausência de uma prova formal de que **todo path runtime relevante** da Git tree foi classificado.

## Achados confirmados

### 1. Declarative presentation registries são uma classe própria

Casos reais na baseline:

- `lib/models/content_display_match_mode.dart` — `storageKey` está corretamente separado, mas `label` é inglês app-owned;
- `lib/models/android_video_renderer_mode.dart` — `storageKey` está separado de `label` e `description`, porém ambos são copy inglesa;
- `lib/models/external_player.dart`;
- `lib/models/windows_external_player.dart`;
- `lib/models/linux_external_player.dart`;
- `lib/models/ios_external_player.dart` — misturam brand/technical data com labels/descriptions app-owned;
- `lib/models/metadata_preferences.dart` — categorias carregam labels ingleses;
- `lib/models/playlist_view_mode.dart` — display labels são resolvidos no model;
- `lib/widgets/launch/**` — id estável + label/subtitle de apresentação.

Consumidores confirmados incluem Settings e Settings Search. Por exemplo, `external_player_settings_page.dart` monta option maps com `mode.storageKey: mode.label`, enquanto `settings_screen.dart` indexa `AndroidVideoRendererMode.values.map((mode) => mode.label)`.

**Risco:** ARB pode chegar a 100% e o app continuar inglês porque o literal não aparece diretamente no widget.

**Correção V10:** Gate D ganha source-to-sink classification para campos/getters/registries; identidade permanece no enum/model, copy vai para mapper localizado.

### 2. CalendarTimeFormat não estava no hotspot manifest V9

`lib/widgets/calendar_display_preferences.dart` contém:

- `Device default`;
- `12-hour`;
- `24-hour`;
- montagem manual de hora 12h;
- literais `AM`/`PM`.

O modo `device` já usa `MaterialLocalizations` corretamente, mas os modos explícitos ainda materializam a apresentação manualmente.

**Correção V10:** enum sem label localizado; mapper para labels; formatter/framework para hora civil explícita 12h/24h.

### 3. StremioTvNowPlaying viola a fronteira domain/presentation

`lib/models/stremio_tv/stremio_tv_now_playing.dart` expõe:

- `progressText` → `Ended` / `Ends at ...`;
- `formatTime` → 12h com `AM/PM` manual.

O consumidor `lib/screens/stremio_tv/widgets/stremio_tv_tuner.dart` renderiza `np?.progressText`.

**Correção V10:** model expõe estado/datas; presentation resolve frase e relógio no locale efetivo.

### 4. Debrify TV stats possui relógio civil e copy manual

`lib/screens/debrify_tv/widgets/stats_tile.dart` monta `HH:mm` diretamente de `DateTime.hour/minute` e combina:

- `Search snapshot`;
- `Queue prepared: ...`;
- `Last search: ...`.

**Correção V10:** ARB + formatter `CIVIL_TIME` + número tipado.

### 5. TV time picker tem chrome app-owned inglês

`lib/widgets/tv_time_picker.dart` já usa `MaterialLocalizations` para AM/PM e para descobrir 12/24h, o que deve ser preservado. Entretanto ainda há:

- `Pick a time`;
- `Up/Down changes · Left/Right moves`;
- `Cancel`;
- `Set`.

**Correção V10:** usar Material labels quando semanticamente equivalentes e AppLocalizations para instrução/copy específica, com testes D-pad.

### 6. Compact metrics são presentation formatting fora da borda visual

Casos confirmados e consumidos pela UI:

- `lib/services/youtube_service.dart::formattedViews` — usa `toStringAsFixed`, `M/K` e inclui `views`;
- `lib/services/reddit_service.dart::formattedScore`;
- `lib/services/lemmy_service.dart::formattedScore`.

Os cards consomem diretamente essas strings.

**Correção V10:** model/service fornece número; presentation usa formatter compact locale-aware e ICU/ARB quando existir substantivo.

### 7. “100% de scan” precisava de closure contra a Git tree

A V9 já exige inventory determinístico e classificação de reachability, mas não expressava a igualdade de conjuntos necessária para provar ausência de path esquecido.

A V10 define:

```text
RuntimeSurfaceUniverse
  == ClassifiedReachable ∪ ExcludedWithEvidence

ClassifiedReachable ∩ ExcludedWithEvidence == ∅
```

Cada path relevante precisa de decisão. Gate Q falha em path novo/alterado sem classificação.

## Regras de implementação resultantes

- não criar Gate R;
- preservar IDs `0,A..Q`;
- reforçar D, E, F, P e Q;
- labels de brand/external/user data não são traduzidos automaticamente;
- heurísticas por nome (`label`, `description`, `formatted*`) geram findings para classificação, não um replace mecânico;
- stable ids/storage keys/protocol tokens/executables permanecem invariantes;
- Settings e Settings Search devem compartilhar o mesmo localized mapper.

## Resultado

A V10 fecha o maior blind spot restante de uma estratégia baseada em hardcoded-string scan: copy indireta e presentation formatting materializados antes do sink.

O plano continua válido somente para o SHA/tree auditado; qualquer mudança upstream volta a passar pelo Gate Q.
