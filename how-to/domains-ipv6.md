---
id: domains-ipv6
title: IPv6 у сервера и режим Cloudflare `Only IPv6` — домен на AAAA-записи
description: "Как опубликовать домен на IPv6 сервера: поле IPv6 address у сервера (Add manually, у Hetzner и Linode из Buy a server AIO заполняет сам, в Edit server поля нет), у Cloudflare DNS-провайдера Records strategy = Only IPv6 (только при Proxied = Yes) — AAAA вместо A, посетители IPv4 и IPv6 идут через прокси Cloudflare; условия, ошибки «DNS provider … uses records strategy "Only IPv6": set IPv6 for server(s) …» и «Records strategy "Only IPv6" requires Proxied = Yes», что происходит с уже заведёнными доменами (переключаются при следующей записи), авторепойнт таких доменов только на серверы группы с IPv6 (нет такого — домен остаётся в Danger), Default IPv4 по умолчанию."
doc_type: how-to
builds: [erp, mtk]
related: [api, server, domains]
language: ru
updated: 2026-11-09
---

# IPv6 у сервера и режим Cloudflare `Only IPv6` — домен на AAAA-записи

**Домен можно опубликовать на IPv6-адрес сервера: у DNS-провайдера Cloudflare поле `Records strategy` = `Only IPv6` ставит домену AAAA-запись на IPv6 сервера вместо A-записи на его IPv4.** Посетители с IPv4 и с IPv6 приходят на Cloudflare, а Cloudflare идёт к серверу по IPv6. Сам сервер продолжает работать с AIO по IPv4: установка, мониторинг сервера и связь агента с AIO не меняются.

По умолчанию стратегия `Default IPv4` — A-записи на IP сервера, как у всех доменов. `Only IPv6` есть только у Cloudflare; у остальных DNS-провайдеров записи всегда A.

Как ещё спрашивают: «домен на IPv6», «AAAA-запись», «IPv6-only», «опубликовать домен по IPv6», «у сервера есть IPv6 — как его использовать».

## Что нужно, чтобы домен работал в режиме `Only IPv6`

**Нужны три вещи: DNS-провайдер Cloudflare с `Proxied` = `Yes` и `Records strategy` = `Only IPv6`, у каждого сервера домена — заполненный `IPv6 address`, и у самой машины — рабочий публичный IPv6.**

1. **Cloudflare в режиме прокси.** Без прокси посетитель без IPv6 до домена не дойдёт, поэтому `Only IPv6` выбирается только при `Proxied` = `Yes` (секция «Как включить `Only IPv6`» ниже).
2. **`IPv6 address` у каждого сервера домена** — и у основного, и у дополнительных (секция «Где у сервера задать `IPv6 address`»). Сервер без IPv6 в форме выбрать можно, но создание домена AIO отобьёт ошибкой, а ручной репойнт пропустит такой домен; авторепойнт сам выбирает только серверы с IPv6 — секция «Репойнт доменов `Only IPv6`».
3. **Публичный IPv6 на машине.** AIO не проверяет, что адрес в поле действительно настроен на сервере: если адрес чужой или у машины нет IPv6, Cloudflare до сервера не достучится и домен перестанет открываться.

## Где у сервера задать `IPv6 address`

**`IPv6 address` — необязательное поле сервера: адрес, на который AIO ставит AAAA-запись доменов в режиме `Only IPv6`.** Подсказка поля: «Server IPv6 — target of the AAAA record when a domain is published via Cloudflare as IPv6 only. Leave empty if the server has no IPv6».

- **`Add manually`** — поле `IPv6 address` рядом с `IP address` (плейсхолдер `e.g. 2a01:4f8::1`): один адрес, без префикса подсети (`/64`). Неверный формат форма не пропускает: `Enter a valid IPv6 address`.
- **`Buy a server` у Hetzner и Linode** — у Hetzner IPv6 у машины включается при заказе, у Linode он есть всегда; AIO записывает адрес сам, когда сервер получает IP. У DigitalOcean, Vultr и остальных провайдеров IPv6 при заказе через AIO не включается — поле остаётся пустым.
- **Уже заведённый сервер** — в окне `Edit server` поля `IPv6 address` нет. Адрес задаётся через Actions API: экшен `Server\Edit` с полем `ipv6` ([how-to/api.md](api.md)). Остальные поля формы берите из текущих значений сервера (`POST /api/v1/actions/data`) и присылайте вместе с `ipv6` — неприсланные `Repoint group` и `Tags` обнулятся.

Адрес сохраняется в стандартной записи (нижний регистр, сжатые нули). У сервера с IPv6 рядом с IP стоит плашка `IPv6` с адресом в подсказке; в таблице серверов есть скрытая колонка `IPv6`, и строка поиска находит сервер по IPv6.

Подробно про добавление сервера — [models/server.md](../models/server.md).

## Как включить `Only IPv6` у DNS-провайдера Cloudflare — `Records strategy`

**`Records strategy` — поле в настройках DNS-провайдера Cloudflare (`Tech → DNS providers`, в MTK — `Settings → DNS providers`): `Default IPv4` (по умолчанию) или `Only IPv6`.** Подсказка поля: «Default IPv4 — A records to the server IPv4. Only IPv6 — AAAA records to the server IPv6 (Cloudflare proxies IPv4 visitors); every server of the domain must have IPv6.»

Пока `Proxied` = `No`, вариант `Only IPv6` в списке неактивен, а если переключить `Proxied` на `No` при выбранном `Only IPv6`, форма сама вернёт `Records strategy` на `Default IPv4` — после сохранения домены провайдера при следующей записи вернутся на A-записи. Сохранить пару `Only IPv6` + `Proxied` = `No` через API тоже нельзя — AIO отвечает `Records strategy "Only IPv6" requires Proxied = Yes: visitors without IPv6 reach the domain only through the Cloudflare proxy`.

Стратегия действует на все домены этого провайдера. Если нужна смесь — IPv4 у одних доменов и IPv6 у других, — заводят два Cloudflare-провайдера: на одном `Default IPv4`, на другом `Only IPv6`, и новые домены добавляют на нужный. У уже заведённого домена DNS-провайдер в интерфейсе не меняется, поэтому его переводят на IPv6 сменой стратегии у его же провайдера. Как подключить Cloudflare и какой ключ нужен — [how-to/domains.md](domains.md).

## Ошибка `uses records strategy "Only IPv6": set IPv6 for server(s)` — у сервера нет IPv6

**Текст `DNS provider <провайдер> uses records strategy "Only IPv6": set IPv6 for server(s) <сервер> (<IP>) or choose another DNS provider` значит, что для домена на провайдере `Only IPv6` выбран сервер без `IPv6 address`.** Ответом на действие он приходит до любых изменений — домен и записи у Cloudflare не трогаются. Строка статуса домена `Repoint failed: DNS provider … uses records strategy "Only IPv6": …` (`Danger`) после авторепойнта — из прежней версии (секция «Репойнт доменов `Only IPv6`»).

Проверка стоит на создании домена (`Buy domain`, `Add manually`, их массовые варианты — ошибка показывается по каждому домену в списке неудач, создание субдомена) и на повторной регистрации домена (`Retry registration`, только ERP). В тексте перечислены все серверы без IPv6.

Ручной `Repoint to a new server` такой домен не трогает, а пропускает с причиной `DNS provider is "Only IPv6", the server has no IPv6` — в итоге после запуска; если пропущены все выделенные домены — `None of the selected domains can be repointed: '<домен>' (DNS provider uses "Only IPv6", selected server has no IPv6)`.

**Что сделать:** задать серверу `IPv6 address` (секция «Где у сервера задать `IPv6 address`»), выбрать другие серверы или другой DNS-провайдер — с `Default IPv4`.

## Что происходит с уже заведёнными доменами при смене стратегии

**Смена `Records strategy` у провайдера уже заведённые домены не трогает: их записи у Cloudflare остаются прежними, пока домен не будет записан заново — ручным репойнтом (`Repoint to a new server`), авторепойнтом при падении сервера, `Balance domains` (ERP) или `Retry registration` (ERP).** При смене стратегии AIO не проверяет, есть ли IPv6 у серверов этих доменов: ошибка всплывёт на первой записи.

При записи AIO ставит запись нужного типа и удаляет записи другого типа на том же имени: домен переключается с A на AAAA (или обратно) целиком, старая A-запись на прежний сервер не остаётся. Новая запись добавляется раньше, чем удаляется старая, поэтому без записи домен не остаётся.

## Репойнт доменов `Only IPv6` — в `Repoint Group` нужен живой сервер с IPv6

**Авторепойнт при падении сервера отправляет домен на провайдере `Only IPv6` только на `Running`-сервер группы с `IPv6 address`.** Если такого нет, домен остаётся в `Danger` с текстом инцидента. В сводке упавшего сервера он считается в общей строке `N left in Danger: no running server to repoint in group <группа>` — даже если другие живые серверы в группе есть, но без IPv6; точная причина — в `Domain logs` домена: `Repoint impossible — left in Danger: no running server with IPv6 to repoint in group <группа> (DNS provider: Only IPv6)`. Поэтому держите IPv6 хотя бы у двух серверов группы, где живут такие домены.

Строка `Repoint failed: DNS provider <провайдер> uses records strategy "Only IPv6": set IPv6 for server(s) <сервер> (<IP>) or choose another DNS provider` после авторепойнта — из прежней версии, когда он мог выбрать сервер без IPv6; встречается на доменах, статус которых с тех пор не менялся. Сейчас такая строка появляется, только если у выбранного сервера убрали `IPv6 address` (или провайдер переключили на `Only IPv6`) уже после постановки репойнта, но до его выполнения.

**Что сделать:** задать IPv6 серверу группы (или поднять сервер с IPv6) и перенести домен вручную (`Repoint to a new server`) на сервер с `IPv6 address`. Общая механика репойнта — [how-to/domains.md](domains.md) → «Repoint при падении сервера».

`Balance domains` на сервере раскладывает домены на провайдере `Only IPv6` только по `Running`-серверам группы с IPv6; если таких нет, домен остаётся на месте, а в его `status_text` пишется `ReBalance skipped: DNS provider uses "Only IPv6", no running server with IPv6 in the group` (статус домена не меняется).

## Проверка домена и мониторинг в режиме `Only IPv6`

**Проверка домена после создания и репойнта и мониторинг доступности работают для домена `Only IPv6` как для обычного.** Строки статуса те же.

Если домен в `Only IPv6` не открывается, а сервер `Running`, первым делом проверьте, что `IPv6 address` сервера совпадает с реальным адресом машины и что на ней работает IPv6.
