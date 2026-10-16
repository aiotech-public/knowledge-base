---
id: visit
title: Visit — концепт (модель)
description: "Visit как сущность пути посетителя в кампании: сессия из URL или cookie, текущий Flow State и накопленные поля; в ERP — независимый Visit Status и списки RMK Audience / RMK Sent Campaigns. Связь с Conversion. Модель-хаб; процесс — visit-lifecycle.md, данные — visit-field.md."
doc_type: model
builds: [erp, mtk]
related: [visit-lifecycle, visit-field, conversion-model, server, domain-model, ui-common, remarketing-campaigns, marketing-flow, debug-with-logs, flow-model, landings, campaign, source]
language: ru
updated: 2026-10-09
---

# Visit — концепт (модель)

> Это **модель-хаб** сущности Visit: что она такое и как связана. Её **процесс** (путь по шагам, где теряется) — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Её **субстрат данных** (поля) — [models/visit-field.md](visit-field.md). Конверсии — [models/conversion-model.md](conversion-model.md).

## Суть в одном абзаце

**Visit — сущность пути посетителя в конкретной кампании: «живой токен», который идёт по Flow и накапливает данные.** При новом входе tracking-ссылка передаёт `cuuid` (кампания) и `suuid` (Source); AIO регистрирует визит с каноническим `visit_uuid`, если агент не продолжил найденную сессию. Сессию восстанавливают из ссылки или cookie. Дальше он движется по шагам флоу по командам AIO (переходы вычисляет AIO, агент на сервере клиента их исполняет — [models/server.md](server.md)), на каждом шаге может быть показан/отфильтрован/потерян, и в итоге может породить конверсию. Почти весь troubleshooting AIO — это «что случилось с визитом»: где он сейчас, по какому переходу пришёл, какие поля заполнены.

## Как это работает на самом деле

### Как идентифицируется визит: cuuid + suuid + cookie → visit_uuid

Агент сначала ищет сессию в URL, затем в **cookie**. При входе с `cuuid`+`suuid` cookie должна соответствовать домену, кампании и Source. Пока подходящая сессия доступна, повторный вход продолжает тот же визит; другой браузер, инкогнито или очистка cookie дают новый вход, если ссылка сама не содержит сессию. Окно уникальности и отличие сессии от fingerprint — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Если сессия не найдена и нет `cuuid`/`suuid`, атрибуцию могут дать Direct Traffic-механизмы (`Default Query` / `Direct Traffic Distribution`).

Как ещё называют: «уники», «уникальные визиты» — про уникальность визита (за сколько браузер продолжает считаться тем же визитом — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md)). Второй ключ к визиту, кроме куки, — зашифрованный слепок сессии в стартовом URL установленной PWA, в клик-ссылке push-уведомления remarketing-кампании и в постоянных ссылках `{{link:s:permanent}}` / `{{link:h:permanent}}`: срока у него нет; такая ссылка может продолжить визит под тем же `visit_uuid` и спустя месяцы, если Visit и связанные объекты доступны, а сессия корректно восстановлена — разбор в [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

### Две ортогональные оси: Flow State (где в пути) vs Visit Status (живость сессии)

**Flow State** показывает положение визита в пути в обоих билдах. В ERP есть и отдельная колонка **Status** — эти две координаты независимы:

- **Текущий Flow State** — *где* визит в графе флоу (на каком шаге стоит). Двигается **переходами** (Transitions): Arrived / Handle / Handle-link / Handle-form / Passed / Rejected / Destination-pushed / Destination-rejected / Destination-interacted / No payload. Это «позиция в пути».

- **Visit Status** — `Wait` / `Live` / `Left` — индикатор активности сессии, **не** позиция. Значение рассчитывается для таблицы по времени последнего доступного события сессии; это не отдельное сохранённое поле-статус визита. Определения и типовые причины — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

Эти оси **не связаны жёстко**: визит может быть `Live`, стоя на Offer, или `Left`, бросив на Preland. Связь «шаги флоу ↔ текущий стейт визита» детально — [models/domain-model.md](domain-model.md) → «Как связаны Flow States и текущий стейт Visit»; статусы.

### Какие поля несёт визит и как они накапливаются по пути флоу

**Visit Fields** хранят данные, которые визит накапливает вдоль пути: URL-параметры входа → данные SDK → значения Fill-шагов. Постбэк тоже может дописать поля визита, если передаёт их в `visit`. Поля — это «состояние» визита, которое читают лэнды, групперы, дистрибуции. Субстрат целиком — [models/visit-field.md](visit-field.md).

Набор колонок таблицы **Visits** не ограничен дефолтными: доступны неархивные поля типов Visit, Source, Campaign, Landing и Destination. В ERP их создают в `Settings → Fields`. Показ и порядок колонок настраиваются кнопкой `Settings` над таблицей → `Table settings` ([reference/ui-common.md](../reference/ui-common.md)). Путь по лэндам и Destination визит копит списками uuid (`landing_uuids`, `destination_uuids`, единая воронка `funnel_uuids`), реферер захода лежит на визите дважды — колонкой и системным полем `Referer`; разбор этих полей и самой таблицы — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

### Что оставляют на визите рассылки: `RMK Audience` (куда визит собран) vs `RMK Sent Campaigns` (путь отправок)

Remarketing-кампании пишут на визит **два разных списка кампаний**, и путать их нельзя. `RMK Audience` (поле `remarketing_campaign_uuids`, категория колонок `Identity`) — в какие remarketing-аудитории визит собран; само попадание в аудиторию ещё ничего не обещает. `RMK Sent Campaigns` (`remarketing_sent_campaign_uuids`, категория `Remarketing`) — путь результатов отправки: кампании, для которых записан результат, включая пропуск по лимиту или отсутствие push-подписки. Запись в этом списке не гарантирует обращение к провайдеру или доставку. Рядом лежат ещё шесть скрытых по умолчанию колонок той же категории, позиционно описывающих тот же путь отправок, — их разбор в [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).

Список аудиторий у визита только пополняется: выбывший визит из `RMK Audience` не вычёркивается, поэтому колонка — история попадания, а не текущий состав аудитории. В какие три точки пути визит собирается в аудиторию — [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md). Где ставится сама пометка кампании и что означает каждая колонка — [how-to/remarketing-campaigns.md](../how-to/remarketing-campaigns.md); модель самого модуля рассылок — [models/marketing-flow.md](marketing-flow.md).

### Конверсия ≠ визит: разные сущности, связь через visit_uuid

Визит и конверсия — **разные сущности**. Конверсия привязывается к визиту по **`visit_uuid`**. Её может создать внешний постбэк, SDK или шаг Flow; постбэк после пуша — один из этих путей. На один визит — много конверсий (single: Lead/Registration; multiple: Sale Status Update). Поэтому «визит есть, конверсии нет» — нормальное промежуточное состояние, а не баг. См. [models/conversion-model.md](conversion-model.md).

### Где визит может потеряться — точки потери (Visit Loss)

На каждом шаге визит может «не дойти»: JS не догрузился, фильтр зарубил (`Rejected`), антифрод срезал перед пушем, дестинейшн отверг, правила Content не подошли (`No payload`). Метрика `Visit Loss` (`Visits Loss %`) показывает долю неквалифицированных визитов: `(Visits − Qualified Visits) / Visits`. Она не суммирует все обрывы на последующих шагах. Поэтому дебаг визита = пройти его путь и найти точку обрыва (см. [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md) → потеря).

## Как Visit связан с остальными сущностями

- **Входит через Campaign (`cuuid`) + Source (`suuid`)** — пара-координата входа.
- **Едет по Flow** — стоит на одном `Flow State`, двигается `Transition`-ами.
- **Видит Landing** на Content-шагах; **пушится в Destination** на выходе.
- **Несёт Visit Fields** (субстрат), **рождает Conversion** (по `visit_uuid`).
- **Пишется в Sessions** (Session Replay), если включён `sessionRecords`.

## Чем Visit отличается от смежных понятий

| Не путать | Разница |
|---|---|
| **Visit vs Conversion** | Visit — *сессия/токен* по флоу. Conversion — *событие*, привязанное к визиту (`visit_uuid`); создаётся постбэком, SDK или шагом Flow. Один визит → много конверсий. |
| **Visit vs Session (Replay)** | Visit — сущность пути/данных. Session — *запись поведения* визита (скроллы/клики) для воспроизведения. Сессия принадлежит визиту. |
| **Visit vs клик / Lead** | Клик в рекламе ≠ визит (визит создаётся, только когда клик доходит до агента и AIO его регистрирует; часть кликов теряется до регистрации, поэтому число визитов и кликов может расходиться). Lead = факт сабмита/пуша, не сам визит. |

**Текущий Flow State vs Visit Status:** первое — позиция в графе; второе — `Wait/Live/Left`, индикатор активности сессии. Оси независимы.

## Подводные камни Visit — частые ошибки в дебаге

- **Повторный клик может продолжить тот же визит**, если агент нашёл подходящую сессию. Идентификация через URL или cookie не равна определению человека: смена атрибуции или утрата сессии может дать новый Visit, а permanent-ссылка — продолжить прежний в другом браузере.
- **`visit_uuid` — твой якорь в дебаге** (Loggable UUID): по нему находят логи, привязанные к визиту; логи Conversion и сообщения рассылки могут иметь собственный Loggable UUID ([how-to/debug-with-logs.md](../how-to/debug-with-logs.md)).

- **`Wait` ≠ «сломалось»** — частый сценарий: первые события SDK ещё не получены.

- **Нет `cuuid`/`suuid` и не найдена сессия** → для нового входа нужна атрибуция через Direct Traffic-механизм.

## Куда идти за деталями Visit — смежные темы

- **Процесс (путь по шагам, где теряется)** → [mechanics/visit-lifecycle.md](../mechanics/visit-lifecycle.md).
- **Субстрат (поля визита)** → [models/visit-field.md](visit-field.md).
- **Конверсии, которые визит рождает** → [models/conversion-model.md](conversion-model.md).
- **Позиция в пути (шаги ↔ стейт)** → [models/flow-model.md](flow-model.md), [models/domain-model.md](domain-model.md).
- **Дебаг конкретного визита** → [how-to/debug-with-logs.md](../how-to/debug-with-logs.md); сессии → [how-to/landings.md](../how-to/landings.md) → Sessions.
- **Вход** → [models/campaign.md](campaign.md), [models/source.md](source.md).
