# Установка панели ScumNeDjin HOST (Wargm v1.1)

Адаптер Wargm-магазина для панели **ScumNeDjin HOST** на выделенном SCUM-сервере (UE4SS).
Wargm v1.0 API отключён провайдером — работает только **v1.1**. В пакете готовый
**`version.dll`** (URL Wargm переключён на v1.1) и конфиги v1.1.

> Установка: распаковал, вписал свои ключи, запустил SCUMServer — и всё.

---

## 0. Что в `ScumNeDjin-host-portable.zip`

```
version.dll                        ← ПАТЧЕННЫЙ бинар (Wargm v1.1). Ставить в Win64/
ScumNeDjin/
├── nedjin.ini                     ← конфиг панели (порт, api_key, bridge_dir)
├── bin/
│   └── e_sqlite3.dll              ← sqlite-зависимый бинар панели
├── configs/
│   ├── wargm-shop.json            ← ВПИШИ СВОИ ProjectId / ApiKey / WargmServerId
│   └── *.json                     ← остальные модули панели (vip, bounty, ...)
├── data/
│   ├── item_data.json             ← каталог предметов (данные, не секреты)
│   ├── es_actor_map.json
│   └── map.png
├── lang/
│   ├── ru.json
│   └── en.json
├── web/
│   ├── app.js  index.html  panel.html ...  ← веб-UI панели (порт 7084)
│   └── (actor/item/spawnable каталоги)
└── INSTALL_RU.txt                 ← краткая памятка (внутри zip)
```

**НЕ входит в пакет** (создаются при первом запуске на хосте):
`ScumNeDjin/state/`, `ScumNeDjin/logs/`, `ScumNeDjin/backups/`.

---

## 1. Куда что положить (каталоги)

`<SCUM>` — корень SCUM-сервера (например `D:\LGSSManagers\Servers\Server1\SCUM`).

| Файл из zip | Куда положить | Зачем |
|---|---|---|
| `version.dll` | `<SCUM>\Binaries\Win64\version.dll` | Wargm URL → v1.1 (заменив старый) |
| `ScumNeDjin/` (весь каталог) | `<SCUM>\Binaries\Win64\ScumNeDjin\` | панель |

Итоговая структура после распаковки:

```
<SCUM>/Binaries/Win64/
├── SCUMServer.exe
├── version.dll                ← из zip (патч Wargm v1.1)
├── UE4SS.dll, UE4SS-settings.ini, ...
└── ScumNeDjin/
    ├── nedjin.ini
    ├── bin/e_sqlite3.dll
    ├── configs/wargm-shop.json   ← вписать ключи
    ├── data/ ...
    ├── lang/ ...
    └── web/ ...
```

> `version.dll` лежит **рядом с `SCUMServer.exe`** (в `Win64/`), внутри
> `ScumNeDjin/` он **не дублируется**. Не копируй `version.dll` в `ScumNeDjin/`.

---

## 2. Вписать свои ключи Wargm

Открой `<SCUM>\Binaries\Win64\ScumNeDjin\configs\wargm-shop.json`:

```jsonc
{
  "Enabled": true,
  "ProjectId": 3856,            // твой project id в Wargm
  "ApiKey": "***",        // твой Wargm API key
  "WargmServerId": 81527,      // ID сервера в Wargm

  // ниже — v1.1, уже прописано, НЕ менять:
  "api_base": "https://api.wargm.ru/v1.1/shop",
  "version": "1.1",
  "sync_endpoint": "/shop/operations",
  "sync_query_status": "pending",
  "operation_success_endpoint": "/shop/operation_success",
  "operation_claim_endpoint": "/shop/operation_claim",
  "operation_cancel_endpoint": "/shop/operation_cancel"
}
```

В пакете `ApiKey`/`ProjectId`/`WargmServerId` пустые — впиши свои.

---

## 3. Запустить

```
старт SCUMServer.exe
```

> Если сервер уже запущен и держит `version.dll` — останови SCUMServer,
> скопируй `version.dll` и `ScumNeDjin/`, запусти обратно.

Панель поднимет HTTP на порту из `nedjin.ini` (по умолчанию **7084**).

Проверки:
- `http://<host>:7084/` — веб-панель
- `http://<host>:7084/api/wargm/pending` — очередь покупок Wargm
- журнал выдачи: `<SCUM>\Binaries\Win64\ScumNeDjin\state\wargm-shop.json` (создаётся при старте)

---

## 4. Контроль здоровья

В `state/wargm-shop.json` (последние записи):

| Строка | Здорово | Ошибка |
|---|---|---|
| sync | `worker-sync-finished result.ok=true fetched=N` | `api-error http=200 msg=Wargm API вернул ошибку.` |
| выдача | `deliver-pending-finished issued=N` | `issued=0` при живых покупках |
| confirm | `confirmResult scanned=N acked=N` | `acked=0` |

Если ошибка: (а) `version.dll` в `Win64/` — v1.1? (б) ключи `wargm-shop.json`?
(в) `WargmServerId` совпадает с Wargm?

---

## 5. Tочки Wargm API v1.1 (используются панелью)

| Метод | Назначение |
|---|---|
| `GET  /v1.1/shop/info` | инфо по магазину |
| `GET  /v1.1/shop/operations?status=pending` | очередь покупок (sync) |
| `POST /v1.1/shop/operation_success` | подтвердить выдачу (тело: `operation_id`) |
| `POST /v1.1/shop/operation_claim` | забрать выдачу |
| `POST /v1.1/shop/operation_cancel` | отменить операцию |

Документация провайдера: https://wargm.ru/docs_api

---

## 6. Запасной вариант — перепатчить version.dll вручную

Готовый `version.dll` уже в zip. Если бинар на хосте «чистый» (URL
`https://api.wargm.ru/v1/`) и нет готового файла:

```python
p = r"<SCUM>\Binaries\Win64\version.dll"
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

> Запускать при **остановленном** `SCUMServer.exe` — иначе файл заблокирован
> (`Permission denied`). После патча перезапусти сервер.
> Проверка: в DLL строка `https://api.wargm.ru/v1.1/`.
