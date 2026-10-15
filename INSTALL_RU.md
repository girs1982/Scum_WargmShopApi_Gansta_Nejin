# ScumNeDjin HOST — пакет с Wargm v1.1

Просто распаковать, поправить 3 числа в конфиге, запустить SCUMServer.

---

## Установка (2 минуты)

### 1. Распаковать

```
D:\SCUM\
└── Binaries\
    └── Win64\          ← сюда распаковать содержимое zip
```

В итоге в `Win64/` появятся/обновятся:

```
Win64/
├── version.dll          ← патч Wargm v1.1 (уже в zip)
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
    ├── bin\
    │   └── e_sqlite3.dll
    ├── configs\
    │   ├── wargm-shop.json    ← ЗДЕСЬ ВПИШИТЬ СВОЁ (см. ниже)
    │   ├── armory-command.json
    │   ├── bounty-hunt.json
    │   ├── bridge-safety.json
    │   ├── character-packs.json
    │   ├── daily-pack.json
    │   ├── discord-log.json
    │   ├── fast-travel.json
    │   ├── home-system.json
    │   ├── private-messages.json
    │   ├── scheduled-events.json
    │   ├── sector-scan.json
    │   ├── server-kill-feed.json
    │   ├── vehicle-rental.json
    │   ├── vip-system.json
    │   ├── welcome-pack.json
    │   └── zombie-infection.json
    ├── data\
    │   ├── es_actor_map.json
    │   ├── item_data.json
    │   └── map.png
    ├── lang\
    │   ├── ru.json
    │   └── en.json
    └── web\
        ├── app.js
        ├── app.css
        ├── index.html
        ├── panel.html
        ├── favicon.ico
        ├── item-icons-manifest.json
        ├── scum-nedjin-profile.jpg
        ├── actor-catalog.json
        ├── actor-catalog-lite.json
        ├── placeable-recipes.json
        ├── spawnable-items.json
        ├── spawnable-items-lite.json
        └── spawnable-vehicles.json
```

### 2. Вписать свои значения

Файл: `Win64\ScumNeDjin\configs\wargm-shop.json`

Вписать только 3 значения:

| Поле | Что вписать | Пример |
|---|---|---|
| `"ProjectId"` | Твой Wargm project ID | `3856` |
| `"ApiKey"` | Твой Wargm API key | `Mb3_eApB_...` |
| `"WargmServerId"` | ID сервера в Wargm | `81527` |

Все остальные v1.1-поля (`api_base`, `sync_endpoint`, `operation_*_endpoint`)
уже прописаны в конфиге — не менять.

`"Enabled": false` (сейчас в пакете) → поставить `true`, когда готов.

### 3. Запустить

```
SCUMServer.exe
```

Панель: `http://<host>:7084/` (порт из `nedjin.ini`).

---

## Проверка

Журнал: `Win64\ScumNeDjin\state\wargm-shop.json` (создаётся при первом старте).

| Строка | Здорово | Ошибка |
|---|---|---|
| sync | `worker-sync-finished result.ok=true fetched=N` | `api-error msg=Wargm API вернул ошибку.` |
| выдача | `deliver-pending-finished issued=N` | `issued=0` |
| confirm | `confirmResult scanned=N acked=N` | `acked=0` |

Признак ошибки `Wargm API вернул ошибку.` → проверь `version.dll` (должен быть
v1.1, `https://api.wargm.ru/v1.1/`) и ключи в `wargm-shop.json`.

---

## Tочки Wargm API v1.1

| Метод | Назначение |
|---|---|
| `GET  /v1.1/shop/info` | инфо по магазину |
| `GET  /v1.1/shop/operations?status=pending` | очередь покупок |
| `POST /v1.1/shop/operation_success` | подтвердить выдачу |
| `POST /v1.1/shop/operation_claim` | забрать выдачу |
| `POST /v1.1/shop/operation_cancel` | отменить операцию |

Документация: https://wargm.ru/docs_api

---

## Что в пакете, что НЕ в пакете

**В пакете:** все файлы панели (бинары, конфиги, web, data, lang), `version.dll`
с патчем v1.1, UE4SS.dll, UE4SS-settings.ini.

**НЕ в пакете (создаются при первом запуске):** `ScumNeDjin/state/`,
`ScumNeDjin/logs/`, `ScumNeDjin/backups/`.

**Не включено намеренно:** `Thumbs.db`, `UE4SS.log`, `crash_*.dmp`, API-ключи
(в `wargm-shop.json` ключи пустые, `nedjin.ini` api_key панели — вписать своё).

---

## Если version.dll не работает (запасной вариант)

Если при старте ошибка `Wargm API вернул ошибку.` — проверь что
`version.dll` содержит v1.1 URL:

```python
import re
data = open(r"Win64\version.dll", "rb").read()
if b"https://api.wargm.ru/v1/" in data and b"https://api.wargm.ru/v1.1/" not in data:
    print("version.dll не патчен — патчим")
    data = bytearray(data)
    i = data.find(b"https://api.wargm.ru/v1/")
    new_url = b"https://api.wargm.ru/v1.1/"
    for k, c in enumerate(new_url):
        data[i+k] = c
    data[i+len(new_url)] = 0
    open(r"Win64\version.dll", "wb").write(bytes(data))
    print("ok")
else:
    print("version.dll уже v1.1")
```

Запускать при остановленном SCUMServer.
