# Scum Wargm Shop API v1.1 (ScumNeDjin host panel)

Адаптер Wargm-магазина для панели **ScumNeDjin HOST** (SCUM dedicated server, UE4SS).
Панель опрашивает Wargm API **v1.1** (operations → delivery → operation_success)
и выдаёт покупки игрокам на сервере.

> Wargm v1.0 API **отключён** провайдером. Работает только v1.1.
> В пакете **уже лежит** запатченный `version.dll` (URL Wargm → v1.1) и конфиги v1.1.

## Быстрый старт

1. **Распакуй** `ScumNeDjin-host-portable.zip` **поверх** каталога `Win64\`
   (там, где лежит `SCUMServer.exe`). Файлы `version.dll`, `UE4SS.dll`,
   `Mods/ScumNeDjinBridge/`, `ScumNeDjin/` встанут на нужные места сами.
2. **Впиши** свой `ProjectId` / `ApiKey` / `WargmServerId` в
   `ScumNeDjin/configs/wargm-shop.json` (3 поля).
3. **Запусти** `SCUMServer.exe`. Панель поднимется на порту из `nedjin.ini`
   (по умолчанию **7084**).
4. **Проверь выдачу**: покупки Wargm приходят в `operations` и
   подтверждаются через `operation_success`.

Подробнее — в `INSTALL_RU.md`.

## Tочки API Wargm v1.1 (используются панелью)

| Метод | Назначение |
|---|---|
| `GET  /v1.1/shop/info` | инфо по магазину |
| `GET  /v1.1/shop/operations?status=pending` | очередь покупок (sync) |
| `POST /v1.1/shop/operation_success` | подтвердить выдачу (тело: `operation_id`) |
| `POST /v1.1/shop/operation_claim` | забрать выдачу |
| `POST /v1.1/shop/operation_cancel` | отменить операцию |

Документация провайдера: https://wargm.ru/docs_api
