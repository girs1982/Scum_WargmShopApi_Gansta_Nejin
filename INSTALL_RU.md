# ScumNeDjin HOST + Wargm v1.1 — установка

Просто распаковать и поправить конфиги. Никакого «клёпанья» по частям.

---

## 1. Распакуй пакет

Распакуй `ScumNeDjin-host-portable.zip` **поверх каталога `Binaries\Win64`**
твоего SCUM-сервера (там, где лежит `SCUMServer.exe`).

В итоге в `Win64\` появятся файлы и папки:

```
Win64\
├── version.dll                     (патч v1.1 — из пакета)
├── UE4SS.dll
├── UE4SS-settings.ini
├── BUILD_INFO.json
├── HOSTING_COMPATIBILITY_RU.md
├── INSTALL_CLIENTS_RU.md
├── INSTALL_RU.txt
├── MANIFEST.txt
├── PANEL_ACCESS.txt
├── RELEASE_NOTES_RU.txt
├── Mods\
│   ├── mods.txt
│   └── ScumNeDjinBridge\
│       ├── enabled.txt
│       ├── data\
│       │   ├── es_actor_map.json
│       │   └── item_data.json
│       └── scripts\
│           └── main.lua
└── ScumNeDjin\
    ├── nedjin.ini
    ├── bin\e_sqlite3.dll
    ├── configs\*.json
    ├── data\*.json
    ├── lang\*.json
    └── web\*
```

> `SCUMServer.exe`, `steam_*.dll`, `tbb12.dll`, `tier0_s64.dll`,
> `vstdlib_s64.dll`, `EOSSDK-Win64-Shipping.dll`, `OpenImageDenoise.dll`,
> `steam_appid.txt` и каталог `Content\` — **не из пакета**, уже в `Win64\`
> как часть самого SCUM-сервера. Не трогай их.

Если `version.dll` уже на хосте и занят запущенным `SCUMServer.exe` — сначала
останови сервер, потом распакуй.

---

## 2. Поправь конфиги

Открой `Win64\ScumNeDjin\configs\wargm-shop.json` и впиши **3 значения**
(все остальные v1.1-поля уже стоят, не менять):

```jsonc
{
  "Enabled": true,
  "ProjectId": 3856,                  // ← твой Wargm project id
  "ApiKey": "***",          // ← твой Wargm API key
  "WargmServerId": 81527,             // ← ID твоего сервера в Wargm

  // v1.1-поля (уже в пакете, НЕ менять):
  "version": "1.1",
  "api_base": "https://api.wargm.ru/v1.1/shop",
  "sync_endpoint": "/shop/operations",
  "sync_query_status": "pending",
  "operation_success_endpoint": "/shop/operation_success",
  "operation_claim_endpoint": "/shop/operation_claim",
  "operation_cancel_endpoint": "/shop/operation_cancel"
}
```

По умолчанию в пакете `Enabled:false`, `ProjectId:0`, `ApiKey:""`,
`WargmServerId:0` — впиши своё.

> `nedjin.ini` (порт панели) и `web/`, `data/`, `lang/` — из пакета, как есть.
> Секретные ключи в пакете отсутствуют (проверено).

---

## 3. Запусти

```
SCUMServer.exe
```

Панель поднимет HTTP на порту из `nedjin.ini` (по умолчанию **7084**).

Проверка:
- `http://<host>:7084/` — веб-панель
- `http://<host>:7084/api/wargm/pending` — очередь покупок Wargm
- журнал: `Win64\ScumNeDjin\state\wargm-shop.json` (создаётся при старте)

---

## Контроль здоровья

Последние записи в `state\wargm-shop.json`:

| Строка | Здорово | Ошибка |
|---|---|---|
| sync | `worker-sync-finished result.ok=true fetched=N` | `api-error msg=Wargm API вернул ошибку.` |
| выдача | `deliver-pending-finished issued=N` | `issued=0` при живых покупках |
| confirm | `confirmResult scanned=N acked=N` | `acked=0` |

Ошибка `Wargm API вернул ошибку.` → проверь: (а) `version.dll` в `Win64\`
— v1.1? (б) ключи в `wargm-shop.json`? (в) `WargmServerId` совпадает с Wargm?

---

## Tочки Wargm API v1.1

| Метод | Назначение |
|---|---|
| `GET  /v1.1/shop/info` | инфо по магазину |
| `GET  /v1.1/shop/operations?status=pending` | очередь покупок (sync) |
| `POST /v1.1/shop/operation_success` | подтвердить выдачу |
| `POST /v1.1/shop/operation_claim` | забрать выдачу |
| `POST /v1.1/shop/operation_cancel` | отменить операцию |

Документация провайдера: https://wargm.ru/docs_api

---

## Что в пакете / чего нет

**В пакете:** `version.dll` (патч v1.1), `UE4SS.dll`, `UE4SS-settings.ini`,
`Mods/ScumNeDjinBridge/` (мост), `ScumNeDjin/` (панель: bin, configs, data,
lang, web, nedjin.ini), справочные `*.md`/`*.txt`/`.json` верхнего уровня.

**Не входит** (создаются при первом запуске): `ScumNeDjin/state/`,
`ScumNeDjin/logs/`, `ScumNeDjin/backups/`.

**Не входит** (часть самого SCUM-сервера, не часть панели): `SCUMServer.exe`,
`steam_*.dll`, `tbb12.dll`, `tier0_s64.dll`, `vstdlib_s64.dll`,
`EOSSDK-Win64-Shipping.dll`, `OpenImageDenoise.dll`, `Content/`.

---

## Запасной вариант — перепатчить version.dll вручную

Готовый `version.dll` (v1.1) уже в пакете. Если вдруг бинар на хосте «чистый»
(URL `https://api.wargm.ru/v1/`) и нужно перепатчить:

```python
p = r"Win64\version.dll"
data = bytearray(open(p, "rb").read())
i = data.find(b"https://api.wargm.ru/v1/")
assert i != -1, "Wargm base URL not found in version.dll"
new_url = b"https://api.wargm.ru/v1.1/"
for k, c in enumerate(new_url):
    data[i + k] = c
data[i + len(new_url)] = 0
open(p, "wb").write(bytes(data))
print("patched:", open(p, "rb").read()[i:i + 30])
```

> При **остановленном** `SCUMServer.exe` (иначе `Permission denied`).
> После — перезапусти сервер. Проверка: в DLL строка `https://api.wargm.ru/v1.1/`.
