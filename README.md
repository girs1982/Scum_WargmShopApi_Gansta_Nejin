# Scum Wargm Shop API v1.1 (ScumNeDjin host panel)

Адаптер Wargm-магазина для панели **ScumNeDjin HOST** (SCUM dedicated server, UE4SS).
Панель опрашивает Wargm API **v1.1** (store/operations → delivery → operation_success)
и выдаёт покупки игрокам на сервере.

> Wargm v1.0 API **отключён** провайдером. Работает только v1.1.
> Пакет содержит бинар с патчем `version.dll` (URL Wargm → `v1.1`) и конфигурации v1.1.

## Состав репозитория

| Файл | Назначение |
|---|---|
| `ScumNeDjin-host-portable.zip` | Портативный установочный пакет панели (bin, configs, web, data, lang, `nedjin.ini`) |
| `version.dll` | Запатченный бинар панели: Wargm URL переключён на `v1.1` (в пакете отсутствует — лежит здесь отдельно) |
| `configs/wargm-shop.json` | Шаблон конфига Wargm-магазина (ключи пусты — вписать свои) |
| `INSTALL_RU.md` | Инструкция по установке на новом сервере |

## Быстрый старт

1. Склепать `version.dll` (патч Wargm v1.1) — см. `INSTALL_RU.md` раздел «Патч version.dll».
2. Развернуть `ScumNeDjin-host-portable.zip` в `SCUM/Binaries/Win64/ScumNeDjin/`.
3. Вписать свой `ProjectId` / `ApiKey` / `WargmServerId` в `ScumNeDjin/configs/wargm-shop.json`.
4. Запустить `SCUMServer.exe`. Панель поднимется на порту из `nedjin.ini` (по умолчанию `7084`).
5. Проверить выдачу: покупки Wargm должны приходить в `operations` и подтверждаться через `operation_success`.

Подробнее — в `INSTALL_RU.md`.

## Точки API Wargm v1.1 (используются панелью)

| Метод | Назначение |
|---|---|
| `GET  /v1.1/shop/info` | инфо по магазину |
| `GET  /v1.1/shop/operations?status=pending` | очередь покупок (sync) |
| `POST /v1.1/shop/operation_success` | подтвердить выдачу (тело: `operation_id`) |
| `POST /v1.1/shop/operation_claim` | забрать (claim) выдачу |
| `POST /v1.1/shop/operation_cancel` | отменить операцию |

Документация провайдера: https://wargm.ru/docs_api
