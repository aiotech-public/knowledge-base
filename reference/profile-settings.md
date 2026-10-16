---
id: profile-settings
title: Profile Settings (настройки профиля пользователя)
description: Окно Profile Settings (меню профиля → Profile settings) — вкладки Profile (аватар, позиции, Available tenants, 2FA, Name / Telegram / Email, пароль) и Appearance (форматы дат/чисел/времени, Component settings с Modal close behavior, Table Style, Fonts, Campaign Flags, Geo Format, профильные картинки, Progress Status). Поля и значения опций; язык интерфейса переключается не здесь, а в шапке.
doc_type: reference
builds: [erp, mtk]
related: [ui-map, registration, architecture, debug-with-logs, notification-center, landings]
language: ru
updated: 2026-10-16
---

# Profile Settings (настройки профиля пользователя)

Модалка настроек **пользователя** (не тенанта). Открывается из меню профиля (аватар, правый верхний угол) → `Profile settings`. Две вкладки: **Profile** и **Appearance**.

> Это персональные настройки конкретного юзера. Настройки тенанта (рабочего пространства) — отдельно, в `Settings → …` (см. [reference/ui-map.md](ui-map.md)).

---

## Как изменить имя, контакты, пароль, 2FA — вкладка Profile

Вкладка `Profile` окна `Profile settings` (подзаголовок `Update your personal details here`) идёт сверху вниз: шапка с аватаром и позициями, список `Available tenants`, блок 2FA `Select a 2FA authentication method`, обязательные поля `Name`, `Telegram` (ник с `@`) и `Email`, поля пароля `Enter the new password` / `Repeat the new password`. Заголовков секций у полей нет.

### Что показано в шапке вкладки Profile — аватар, позиции, `Available tenants`

Шапка: аватар, имя, email и позиции (Position) юзера; у Owner / Admin вместо позиций стоит `Super access`. Чтобы сменить аватар, навести курсор на картинку → `Upload photo` → окно `Upload new avatar` (drag-and-drop или файловый браузер) → `Next`. `Next` только подставляет картинку в окно — на аккаунт аватар записывается при сохранении профиля (`Save` или галочка в поле).

Под шапкой — `Available tenants` со счётчиком: доступные тенанты чипами, текущий выделен цветом, у тенанта с заданным билдом подписан билд. Список только для просмотра — тенант переключается в меню профиля ([reference/ui-map.md](ui-map.md)).

### Как сменить имя, Telegram, email и пароль — галочка `Click to save` и `Save`

Поля `Name`, `Telegram` и `Email` обязательные; пароль меняется парой `Enter the new password` / `Repeat the new password` (требования к паролю — [how-to/registration.md](../how-to/registration.md)). Кнопка `Save` внизу сохраняет все изменения окна и закрывает его, `Cancel` закрывает без сохранения.

Если изменить `Telegram` или `Email`, в поле появляется галочка с подсказкой `Click to save`. Клик по ней сохраняет все изменения окна и не закрывает его: тост `Saved` / `Your changes have been successfully saved`, у галочки подсказка `Saved`. Смена значения в `Telegram` снимает привязку аккаунта к боту — `/setup2fa` нужно пройти заново ([how-to/registration.md](../how-to/registration.md)); при включённой 2FA смена контактов просит код (секция «Смена Telegram или Email при включённой 2FA» в этом доке).

### Блок 2FA `Select a 2FA authentication method` — Telegram или Google Authenticator

В блоке две карточки: `Telegram` (`Use Telegram bot for verification`) и `Google Authenticator` (`Use authenticator apps (TOTP)`); кода на email или по SMS нет. При включённой 2FA QR и секрет Google Authenticator не показываются, а способ меняется только через `Disable 2FA` кодом включённого способа. Пошаговая процедура включения, выключения и смены способа — [context/architecture.md](../context/architecture.md) (§ «Как включить 2FA» и «Как выключить 2FA или сменить способ»); если код из Google Authenticator не подходит (`The entered code is incorrect`) — там же, § «Код из Google Authenticator не подходит».

### Смена Telegram или Email при включённой 2FA — нужен код

Если 2FA включена, при сохранении нового `Telegram` или `Email` (галочкой в поле или кнопкой `Save`) сначала открывается окно `Two factor authentication` с кодом текущего способа 2FA. Для Telegram это 4 цифры от `AIO-tech-bot`: код уходит сам при открытии окна в ещё привязанный, то есть старый, Telegram, повторно — кнопкой `Resend code`. Для приложения-аутентификатора — 6 цифр (`Enter the code from the 2FA App`). Имя, пароль, аватар и `Appearance` сохраняются без кода, если в том же сохранении не меняются Telegram или Email. Как ещё спрашивают: «сменил ник в Telegram», «поменял почту», «просит код при смене контактов».

### После смены Telegram-ника перестали приходить коды и уведомления — привязка к боту сброшена

Смена Telegram-ника сбрасывает привязку аккаунта к боту, и это не зависит от 2FA: личные сообщения бота и коды 2FA перестают приходить, пока не отправить `/setup2fa` с нового аккаунта Telegram ([how-to/registration.md](../how-to/registration.md)). Групповые чаты, привязанные командой `/bind`, это не затрагивает ([mechanics/notification-center.md](../mechanics/notification-center.md)).

При подтверждении смены Telegram 2FA выключается при любом способе, в том числе при Google Authenticator. `/setup2fa` с нового аккаунта сам привязывает чат и включает 2FA обратно тем способом, который стоял на аккаунте. Пока чат не привязан, `Enable 2FA` для способа Telegram показывает `2FA has not been enabled` и кнопку `Go to bot`. Смена одного только Email 2FA не выключает. Если после ввода кода сохранение не прошло (например, ник занят другим аккаунтом), Telegram остаётся прежним, а 2FA — выключенной: проверьте блок 2FA и включите её заново ([context/architecture.md](../context/architecture.md), § «Как включить 2FA»). Как ещё спрашивают: «2FA выключилась сама».

## Как настроить форматы дат, шрифты, отображение таблиц — вкладка Appearance

Вкладка `Appearance` кастомизирует отображение и форматы данных в интерфейсе (форматы дат/чисел, стиль таблиц, шрифты, гео-флаги). Секции:

- **`Locale settings`**:
  - `Weekdays start` — начало недели: `Monday` или `Sunday`.
  - `Numbers locale` — формат чисел.
  - `Location` — таймзона пользователя.
  - `Dates locale` — формат дат.
- **`Table quick filters settings`**:
  - `Selects behaviour` — как ведут себя фильтры-селекты в таблицах.
  - `Icons showing` — показывать ли иконки в быстрых фильтрах.
- **`Component settings`**:
  - `Modal close behavior` — как закрывается модальное окно: `Backdrop click` (дефолт) или `Cancel click`. Разбор — подсекция ниже.
  - `Domain amounts` — что показывать в счётчике доменов.
- **`Table style`** — строки таблиц: `Striped` (полосатые) или `Plain`.
- **`Fonts`**:
  - `Font class` — `Monospaced` или `Sans`.
  - `Font style` — `Bold` или `Regular`.
- **`Campaign flags`** — гео-флаги в таблицах: `Show` / `Hide`.
- **`Campaign assigned colors`** — показывать ли назначенные кампаниям цвета.
- **`Geo format`** — как показывать гео в таблицах: `Country flag` или `Country name`.
- **`User profile picture`** — аватарки юзеров в таблицах: `Show` / `Hide`.
- **`Progress status`** — отображение прогресс-статуса в таблицах: `Progress bar` или `Number`.

### Как `Modal close behavior` спасает несохранённые правки

`Backdrop click` (по умолчанию) закрывает диалог кликом по затемнению вокруг него, `Cancel click` — только своей кнопкой закрытия или отмены, а также клавишей `Esc`: `Esc` закрывает диалоги при любом значении настройки. Настройка своя у каждого пользователя и действует на все диалоги, которые не задают поведение сами.

Практический смысл — диалоги, которые сами не спрашивают о несохранённых правках: на дефолте случайный клик мимо окна закрывает форму вместе с правками, а `Cancel click` это исключает. `Manage landing` спрашивает сам — [how-to/landings.md](../how-to/landings.md). Окно подтверждения 2FA настройку игнорирует — оно всегда закрывается только своей кнопкой.

---

## Где переключить язык интерфейса — кнопка в шапке, не в Profile Settings

Язык интерфейса переключается **в верхней панели**, а не в настройках профиля: сразу справа от часов (дата, время, смещение таймзоны) стоит кнопка с флагом и кодом текущего языка (`EN`, `RU`, …), она открывает список из восьми языков — `English`, `Русский`, `Українська`, `Español`, `Italiano`, `Tiếng Việt`, `Português`, `中文`. У всех, кроме английского, рядом стоит бейдж `ALPHA`. Во вкладке `Appearance` настраиваются только форматы данных (`Locale settings` — начало недели, форматы чисел и дат, таймзона), поля «язык» там нет.

Выбранный язык хранится **в браузере**, а не на аккаунте: в другом браузере, на другом устройстве или после очистки данных сайта интерфейс снова откроется на языке браузера (если он из этих восьми) либо на английском.

Тот же переключатель записывает на аккаунт **язык уведомлений** — на нём приходят уведомления в Telegram, Slack, колокольчике и сообщения бота. Кнопка стоит и на экране входа, но там она меняет только интерфейс: на аккаунт язык уходит только из залогиненного интерфейса. Что переводится, что остаётся английским и почему уведомления бывают английскими при русском интерфейсе — [mechanics/notification-center.md](../mechanics/notification-center.md).

---

## Смежные темы

- [reference/ui-map.md](ui-map.md) — где `Profile settings` в меню профиля.
- [context/architecture.md](../context/architecture.md) — 2FA через `AIO-tech-bot`, включение и роль двухфакторки.
