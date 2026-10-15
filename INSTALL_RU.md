# Установка панели ScumNeDjin HOST (Wargm v1.1)

Адаптер Wargm-магазина для панели **ScumNeDjin HOST** на выделенном SCUM-сервере (UE4SS).
Включает **патч `version.dll`** (переключение Wargm API на v1.1) и конфигурации v1.1.

> Wargm v1.0 отключён провайдером, работает только v1.1.
> Пакет содержит `bin/ configs/ web/ data/ lang/ nedjin.ini` + отдельный файл `version.dll`.

---

## 0. Что в комплекте

| Файл / папка | Куда ставить |
|---|---|
| `version.dll` | `<SCUM>/Binaries/Win64/` |
| `ScumNeDjin/` (из `ScumNeDjin-host-portable.zip`) | `<SCUM>/Binaries/Win64/ScumNeDjin/` |
| `configs/wargm-shop.json` | вписать свои ключи в `<SCUM>/Binaries/Win64/ScumNeDjin/configs/wargm-shop.json` |

`<SCUM>` — корень каталога SCUM-сервера, например `D:\LGSSManagers\Servers\Server1\SCUM`.

---

## 1. Патч `version.dll` (Wargm → v1.1)

Панель `ScumNeDjin HOST` хранит базовый URL Wargm в `version.dll`. В дефолтном
бинаре он зашит как `https://api.wargm.ru/v1/` — отсюда и ошибка
`"Wargm API вернул ошибку."` (v1.0 отключён).

### Вариант A — взять готовый (рекомендуется)

В репозитории лежит **запатченный** `version.dll`. Просто скопируй его:

```
<репо>/version.dll  →  <SCUM>/Binaries/Win64/version.dll
```

### Вариант B — патчить самому (если нет файла из репо)

Строка в `version.dll`: `https://api.wargm.ru/v1/` (23 байта) заменяется на
`https://api.wargm.ru/v1.1/` (25 байт). Два свободных NUL после строки
покрывают доп. 2 байта — размер DLL не меняется.

Python (запускать **при остановленном** `SCUMServer.exe`, иначе файл занят):

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
print("patched:", open(p, "rb").read()[i : i + 30])
```

Проверка: `grep -a "wargm.ru/v1.1/" version.dll` → должна найтись строка
`https://api.wargm.ru/v1.1/`.

> `SCUMServer.exe` держит `version.dll` заблокированным, пока запущен.
> Останавливай сервер **до** патча/копирования, иначе `Permission denied`.

---

## 2. Развернуть портативный пакет

```bash
cd <SCUM>/Binaries/Win64
# распаковать содержимое ScumNeDjin/ в Win64/ScumNeDjin/
# (папку bin/ data/ lang/ web/ configs/ nedjin.ini из zip)
```

Ожидаемая структура после:

```
<SCUM>/Binaries/Win64/
├── version.dll              # patched (v1.1)
└── ScumNeDjin/
    ├── nedjin.ini
    ├── bin/  configs/  data/  lang/  web/
    └── ...
```

---

## 3. Настроить Wargm (обязательно)

Открой `<SCUM>/Binaries/Win64/ScumNeDjin/configs/wargm-shop.json` и впиши **свои**:

```jsonc
{
  "Enabled": true,
  "ProjectId": 3856,                              // твой project id в Wargm
  "ApiKey": "***",            // твой Wargm API key
  "WargmServerId": 81527,                        // ID сервера в Wargm

  // ниже — v1.1, уже прописаны, не менять:
  "api_base": "https://api.wargm.ru/v1.1/shop",
  "version": "1.1",
  "sync_endpoint": "/shop/operations",
  "sync_query_status": "pending",
  "operation_success_endpoint": "/shop/operation_success",
  "operation_claim_endpoint": "/shop/operation_claim",
  "operation_cancel_endpoint": "/shop/operation_cancel",
  ...
}
```

Ключ в пакете намеренно **пустой** — не разглашай чужие API-ключи.

---

## 4. Запустить

```
старт SCUMServer.exe
```

Панель поднимет HTTP на порту из `nedjin.ini` (по умолчанию **7084**).
Проверка: `http://<host>:7084/` — веб-панель;
`http://<host>:7084/api/wargm/pending` — очередь покупок Wargm.

---

## 5. Контроль выдачи

Живой журнал выдачи: `<SCUM>/Binaries/Win64/ScumNeDjin/state/wargm-shop.json`
(создаётся при первом запуске; в пакете его нет).

Признаки здоровья:

```
worker-sync-finished result.ok=true fetched=2
deliver-pending-finished issued=2
confirmResult scanned=N acked=N
```

Признаки ошибки:

```
api-error http=200 msg=Wargm API вернул ошибку.
```

Если ошибка — проверить: (a) `version.dll` на v1.1, (b) `ApiKey`/`ProjectId`,
(c) `WargmServerId` совпадает с тем, что указан в Wargm.

---

## Tочки Wargm API v1.1

| Метод | Назначение |
|---|---|
| `GET  /v1.1/shop/info` | инфо по магазину |
| `GET  /v1.1/shop/operations?status=pending` | очередь покупок (sync) |
| `POST /v1.1/shop/operation_success` | подтвердить выдачу (тело: `operation_id`) |
| `POST /v1.1/shop/operation_claim` | забрать выдачу |
| `POST /v1.1/shop/operation_cancel` | отменить операцию |

Документация: https://wargm.ru/docs_api
