---
id: visit
title: Visit — концепт (модель)
description: Что такое визит в AIO как сущность — одна браузерная сессия в кампании (cuuid+suuid+cookie), живой инстанс, идущий по Flow и накапливающий поля; две ортогональные оси (Flow State vs Visit Status), связь с конверсией, какие списки рассылок лежат на визите (RMK Audience — в какие аудитории визит собран, RMK Sent Campaigns — путь отправок). Модель-хаб; процесс — visit-lifecycle.md, данные — visit-field.md.
doc_type: model
builds: [erp, mtk]
related: [visit-lifecycle, visit-field, conversion-model, server, domain-model, remarketing-campaigns, marketing-flow, debug-with-logs, flow-model, landings, campaign, source]
language: ru
updated: 2026-08-11
---

# Visit — концепт (модель)

> Это **модель-хаб** сущности Visit: что она такое и как связана. Её **процесс** (путь по шагам, где теряется) — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Её **субстрат данных** (поля) — [models/visit-field.md](visit-field.md). Конверсии — [models/conversion-model.md](conversion-model.md).

## Суть в одном абзаце

**Visit — это одна браузерная сессия в конкретной кампании, и одновременно «живой токен», который идёт по Flow и накапливает на себе данные.** Визит создаётся, когда tracking-ссылка с `cuuid` (кампания) + `suuid` (source) прилетает на агент и AIO его регистрирует; идентифицируется этой парой + cookie в браузере; получает `visit_uuid` — свой канонический id. Дальше он движется по шагам флоу по командам AIO (переходы вычисляет AIO, агент на сервере клиента их исполняет — [models/server.md](server.md)), на каждом шаге может быть показан/отфильтрован/потерян, и в итоге может породить конверсию. Почти весь troubleshooting AIO — это «что случилось с визитом»: где он сейчас, по какому переходу пришёл, какие поля заполнены.

## Как это работает на самом деле

### Как идентифицируется визит: cuuid + suuid + cookie → visit_uuid

Визит опознаётся **парой `cuuid`+`suuid`** (в какую кампанию и через какой source) плюс **cookie** в браузере посетителя. Новый визит — другой браузер / инкогнито / очистка кук. Тот же браузер остаётся тем же визитом, пока возвращается; окно уникальности и полная механика идентификации, дубли (сессия vs fingerprint) — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Нет `cuuid`/`suuid` → Direct Traffic (`Default Query` / `Direct Traffic Distribution`).

Как ещё называют: «уники», «уникальные визиты» — про уникальность визита (за сколько браузер продолжает считаться тем же визитом — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md)).

### Две ортогональные оси: Flow State (где в пути) vs Visit Status (живость сессии)

Это главный концепт, который снимает кучу путаницы. У визита **две независимые координаты**:

- **Текущий Flow State** — *где* визит в графе флоу (на каком шаге стоит). Двигается **переходами** (Transitions): Arrived / Handle / Handle-link / Handle-form / Passed / Rejected / Destination-pushed / Destination-rejected / Destination-interacted / No payload. Это «позиция в пути».
- **Visit Status** — `Wait` / `Live` / `Left` — *живость сессии*, **не** позиция. Это **производный концепт записи сессии, а не хранимая колонка визита** — у визита нет поля-статуса, значения выводятся из состояния session-record (единственный серверный сигнал живости — отметка последнего события сессии). Определения и типовые причины — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

Эти оси **не связаны жёстко**: визит может быть `Live`, стоя на Offer, или `Left`, бросив на Preland. Связь «шаги флоу ↔ текущий стейт визита» детально — [models/domain-model.md](domain-model.md) → «Как связаны Flow States и текущий стейт Visit»; статусы.

### Какие поля несёт визит и как они накапливаются по пути флоу

Всё, что AIO знает о визите, лежит в его **полях** (visit fields). Значение **аккретит вдоль пути**: Source кладёт URL-параметры → SDK пишет рантайм → Fill-шаги дописывают → постбэк добавляет результат конверсии. Поля — это «состояние» визита, которое читают лэнды, групперы, дистрибуции. Субстрат целиком — [models/visit-field.md](visit-field.md).

Набор колонок в `Tracker → Visits` **не ограничен дефолтными** — любое поле, заведённое в `Settings → Fields`, автоматически становится доступной колонкой в Visits (показ/порядок — через `Presets → Customize table view`). Подробнее про таблицу и колонки — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

### Что оставляют на визите рассылки: `RMK Audience` (куда визит собран) vs `RMK Sent Campaigns` (путь отправок)

Remarketing-кампании пишут на визит **два разных списка кампаний**, и путать их нельзя. `RMK Audience` (поле `remarketing_campaign_uuids`, категория колонок `Identity`) — в какие remarketing-аудитории визит собран; само попадание в аудиторию ещё ничего не обещает. `RMK Sent Campaigns` (`remarketing_sent_campaign_uuids`, категория `Remarketing`) — путь отправок: кампании, у которых дело дошло до попытки отправить этому визиту. Рядом лежат ещё шесть скрытых по умолчанию колонок той же категории, позиционно описывающих тот же путь отправок, — их разбор в [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

Список аудиторий у визита только пополняется: выбывший визит из `RMK Audience` не вычёркивается, поэтому колонка — история попадания, а не текущий состав аудитории. В какие три точки пути визит собирается в аудиторию — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Где ставится сама пометка кампании и что означает каждая колонка — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md); модель самого модуля рассылок — [models/marketing-flow.md](marketing-flow.md).

### Конверсия ≠ визит: разные сущности, связь через visit_uuid

Визит и конверсия — **разные сущности**. Конверсия приходит **постбэком** (позже пуша) и привязывается к визиту по **`visit_uuid`**. На один визит — много конверсий (single: Lead/Registration; multiple: Sale Status Update). Поэтому «визит есть, конверсии нет» — нормальное промежуточное состояние, а не баг. См. [models/conversion-model.md](conversion-model.md).

### Где визит может потеряться — точки потери (Visit Loss)

На каждом шаге визит может «не дойти»: JS не догрузился, фильтр зарубил (`Rejected`), антифрод срезал перед пушем, дестинейшн отверг, правила Content не подошли (`No payload`). Сумма таких потерь — метрика `Visit Loss`. Поэтому дебаг визита = пройти его путь и найти точку обрыва (см. [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) → потеря).

## Как Visit связан с остальными сущностями

- **Входит через Campaign (`cuuid`) + Source (`suuid`)** — пара-координата входа.
- **Едет по Flow** — стоит на одном `Flow State`, двигается `Transition`-ами.
- **Видит Landing** на Content-шагах; **пушится в Destination** на выходе.
- **Несёт Visit Fields** (субстрат), **рождает Conversion** (по `visit_uuid`).
- **Пишется в Sessions** (Session Replay), если включён `sessionRecords`.

## Чем Visit отличается от смежных понятий

| Не путать | Разница |
|---|---|
| **Visit vs Conversion** | Visit — *сессия/токен* по флоу. Conversion — *событие*, пришедшее обратно постбэком и привязанное к визиту (`visit_uuid`). Один визит → много конверсий. |
| **Visit vs Session (Replay)** | Visit — сущность пути/данных. Session — *запись поведения* визита (скроллы/клики) для воспроизведения. Сессия принадлежит визиту. |
| **Visit vs клик / Lead** | Клик в рекламе ≠ визит (визит создаётся, только когда клик доходит до агента и AIO его регистрирует; часть кликов теряется — норма, визитов в AIO меньше, чем кликов в рекламном кабинете). Lead = факт сабмита/пуша, не сам визит. |
| **Текущий Flow State vs Visit Status** | Первое — *где* в пути (позиция в графе). Второе — `Wait/Live/Left`, *живость* сессии. Ортогональны. |

## Подводные камни Visit — частые ошибки в дебаге

- **Тот же браузер по той же кампании = тот же визит** — повторные клики не создают новый визит: он привязан к человеку, а не к клику (by design). Новый визит — другой браузер/инкогнито.
- **`visit_uuid` — твой якорь в дебаге** (Loggable UUID): по нему собираются все события визита ([how-to/debug-with-logs.md](../how-to/debug-with-logs.md)).
- **`Wait` ≠ «сломалось»** — часто визит просто ждёт первые ивенты от SDK (запись сессии не началась; *уточните у поддержки*).
- **Нет `cuuid`/`suuid`** → визит не привяжется без Direct Traffic-механизма.

## Куда идти за деталями Visit — смежные темы

- **Процесс (путь по шагам, где теряется)** → [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).
- **Субстрат (поля визита)** → [models/visit-field.md](visit-field.md).
- **Конверсии, которые визит рождает** → [models/conversion-model.md](conversion-model.md).
- **Позиция в пути (шаги ↔ стейт)** → [models/flow-model.md](flow-model.md), [models/domain-model.md](domain-model.md).
- **Дебаг конкретного визита** → [how-to/debug-with-logs.md](../how-to/debug-with-logs.md); сессии → [how-to/landings.md](../how-to/landings.md) → Sessions.
- **Вход** → [models/campaign.md](campaign.md), [models/source.md](source.md).
