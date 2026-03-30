# Workflow: Database Sync & Updates

## Forms Involved

| Form | Role |
|---|---|
| `frReproServer` | Manual sync with web server (ReproServer.asp) |
| `frSyncDBProgress` | Progress/log dialog for local DB schema updates |
| `frReproUpdate` | Downloads and applies application updates via HTTP |
| `frLanguages` | Download and select UI language packs from server |
| `clsReproServer.cls` | HTTP/XML client class |
| `mdlSyncDB.bas` | DB schema migration logic |
| `ovt/ovt_settings.bas` | Device ID init, host config |

## Sync Types

Defined in [project/ovt/ovt_common.bas:5-8](../../project/ovt/ovt_common.bas#L5-L8):

| Constant | Value | Target |
|---|---|---|
| `SYNC_WEB` | 1 | Web server (ReproServer.asp) |
| `SYNC_DSK` | 2 | Desktop (this app) |
| `SYNC_PKT` | 3 | PocketPC device |
| `SYNC_PLM` | 4 | Palm device |

## Flow: Web Server Sync (frReproServer)

```
frReproServer
  cbOperation dropdown — select sync operation
  ListView (columns: Records, Operation, New, Updated, Deleted, Transfer Status)
  btAccount → configure username/password
  btStart → clsReproServer.Init(ls, username, password)
           → clsReproServer.RequestXML(xmlDoc)  [synchronous POST]
               POST to: http://<REPRO_HOST>/ReproServer.asp?r=<random>
               Body: XML document with operation + data
               Response: XML with record counts per table
           → on success: parse response XML, update ListView rows
           → on failure: RaiseEvent failed(status, message) → shown in txtReport
  btStop → abort (sets FailStatus)
  txtReport → Courier New log of raw response text
```

`clsReproServer` raises two events (consumed by `frReproServer`):
- `Report(Message)` — progress text
- `failed(status, Message)` — error with HTTP status code

Cache-busting: every request URL includes `?r=<Rnd()*100000>` — see [project/clsReproServer.cls:31](../../project/clsReproServer.cls#L31).

Error detection: checks both `http.status <> 200` AND whether response XML root node is `<ERROR>` — see [project/clsReproServer.cls:44-56](../../project/clsReproServer.cls#L44-L56).

## Flow: Database Schema Update (frSyncDBProgress)

```
frSyncDBProgress ("Database Update")
  Shown when app detects local DB schema is behind current version
  ProgressBar pbComplete + txtLog (locked, multiline, scrollable)
  Label: "Your database is being updated. Please wait..."
  btClose (labeled "Continue") — enabled only when update completes
  mdlSyncDB.bas — runs ALTER TABLE / CREATE TABLE statements sequentially
                  updates txtLog and pbComplete as each step completes
```

## Flow: Application Update (frReproUpdate)

```
frReproUpdate ("Repro Manager Update")
  Uses MSINET.OCX (Inet control) for HTTP download
  ProgressBar pbComplete
  lblStatus — shows download %
  btStop — cancels download
  On complete: launches downloaded installer
```

## Flow: Language Packs (frLanguages)

```
frLanguages ("Languages")
  Lists available language files in fmLang frame
  btGetSrvLang ("Check For Updates") — fetches language list from server
  cmdSave → applies selected language (writes to GetVar/SetVar registry key)
  cmdCancel → discards selection
```

## Device Identity

On first launch, a unique device ID is generated and stored in the registry:
```
ID_DVC = get_id(12, 15) * 10 + SYNC_DSK
SetVar "ID_DVC", ID_DVC
```
See [project/ovt/ovt_settings.bas:17-20](../../project/ovt/ovt_settings.bas#L17-L20). This ID is sent with every sync request to identify the originating device and prevent ID collisions across sync targets.

## Host Configuration

`OVT_HOST = "ovtvet.com"` set in `Ovt_InitSettings` — see [project/ovt/ovt_settings.bas:10](../../project/ovt/ovt_settings.bas#L10).
`REPRO_HOST` is a separate constant used by `clsReproServer`.
