---
id: profile-settings
title: Profile Settings (настройки профиля пользователя)
description: Окно Profile Settings (меню профиля → Profile settings) — вкладки Profile (Name, Contact, Password, 2FA) и Appearance (форматы дат/чисел/времени, Component settings с Modal close behavior, Table Style, Fonts, Campaign Flags, Geo Format, профильные картинки, Progress Status). Поля и значения опций.
doc_type: reference
builds: [erp, mtk]
related: [ui-map, architecture, debug-with-logs]
language: ru
updated: 2026-08-11
---

# Profile Settings (настройки профиля пользователя)

Модалка настроек **пользователя** (не тенанта). Открывается из меню профиля (аватар, правый верхний угол) → `Profile settings`. Две вкладки: **Profile** и **Appearance**.

> Это персональные настройки конкретного юзера. Настройки тенанта (рабочего пространства) — отдельно, в `Settings → …` (см. [reference/ui-map.md](ui-map.md)).

---

## Как изменить имя, контакты, пароль, 2FA — вкладка Profile

Вкладка `Profile` показывает имя пользователя и список доступных ему тенантов (текущий выделен). Здесь же — смена имени, контактов, пароля и двухфакторки. Секции:

- **Personal Information** — поле `Name` (имя пользователя). Меняется: ввести новое значение → `Save`.
- **Contact** — `Telegram username` + email-адрес.
- **Password** — смена пароля через поля `Enter the new password` и `Repeat the new password` → `Save`.
- **2FA** — двухфакторка. Включается через `AIO-tech-bot` в Telegram **или** через Google Authenticator (привязка к Telegram для 2FA больше не обязательна). Пошаговая процедура включения — в [context/architecture.md](../context/architecture.md) (§ AIO-tech-bot и 2FA).

**Profile picture** (аватар) — навести курсор на картинку → кнопка `Upload new photo` → попап `Upload new avatar` (drag-and-drop или файловый браузер) → `Next` (сохранить загруженный файл).

Глобальная кнопка `Save` внизу окна сохраняет все внесённые изменения.

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

`Backdrop click` (по умолчанию) закрывает диалог кликом по затемнению вокруг него, `Cancel click` — только своей кнопкой закрытия или отмены. Настройка своя у каждого пользователя и действует на все диалоги, которые не задают поведение сами.

Практический смысл — тяжёлые формы вроде `Manage landing`: на дефолте случайный клик мимо окна закрывает форму вместе с несохранёнными правками, а `Cancel click` это исключает. Окно подтверждения 2FA настройку игнорирует — оно всегда закрывается только своей кнопкой.

---

## Смежные темы

- [reference/ui-map.md](ui-map.md) — где `Profile settings` в меню профиля.
- [context/architecture.md](../context/architecture.md) — 2FA через `AIO-tech-bot`, включение и роль двухфакторки.
