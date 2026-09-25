# Matriz de testes i18n

## Locales obrigatórios

- `en`
- `pt_BR`
- pseudo-LTR expandido
- pseudo-RTL (debug)

## Superfícies

| Superfície | Unit | Widget | Golden | Manual TV | Native |
|---|---:|---:|---:|---:|---:|
| App root/bootstrap | ✅ | ✅ | opcional | ✅ | |
| Profiles | ✅ | ✅ | ✅ | ✅ | |
| Settings | ✅ | ✅ | ✅ | ✅ | |
| Settings Search | ✅ | ✅ | | ✅ | |
| Onboarding | ✅ | ✅ | ✅ | ✅ | |
| Home/Search/Details | | ✅ | ✅ | ✅ | |
| Sources | ✅ | ✅ | ✅ | ✅ | |
| Player Flutter | ✅ | ✅ | ✅ | ✅ | |
| Android native player | ✅ | | | ✅ | ✅ |
| IPTV | ✅ | ✅ | ✅ | ✅ | |
| Debrify TV | ✅ | ✅ | ✅ | ✅ | |
| Stremio TV | ✅ | ✅ | ✅ | ✅ | |
| Sync/Backup/Recovery | ✅ | ✅ | | ✅ | |
| Formatters | ✅ | | | | |
| Web/Desktop metadata | ✅ | | | | |

## Regressões críticas

- locale nunca altera metadata/audio/subtitle prefs;
- locale nunca entra em IDs persistidos;
- WebDAV não sobrescreve idioma;
- troca de profile não altera idioma;
- player nativo recebe o override;
- strings longas não quebram D-pad;
- todos os locales shipping têm 100% das ARB keys.
