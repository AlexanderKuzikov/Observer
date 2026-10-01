<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/Status-Research-blue.svg">
  <img alt="Report" src="https://img.shields.io/badge/Report-2026--10--01-orange.svg">
  <img alt="License" src="https://img.shields.io/badge/License-Apache_2.0-blue.svg">
</p>

<h1 align="center">Observer</h1>
<p align="center">Исследовательский репозиторий: аудит инструментов и внешних API</p>

---

Исследование инструментария разработчика на 2026 год с проверкой по первоисточникам.
Фокус: MCP-экосистема, SEO и разработка сайтов, российские госреестры и
бухгалтерский контур, стек разработки, обработка документов и скрапинг.

Все данные проверены **2026-10-01**. Где проверка не удалась — помечено явно,
чтобы отличить факт от вывода.

## Что внутри

| Документ | О чём |
|---|---|
| [`docs/RESEARCH-2026-10.md`](docs/RESEARCH-2026-10.md) | Главный отчёт: сводка, матрица приоритетов, план действий |
| [`docs/research/01-mcp.md`](docs/research/01-mcp.md) | MCP-экосистема, спецификация 2026-07-28, риски, что ставить |
| [`docs/research/02-seo.md`](docs/research/02-seo.md) | SEO: API Яндекса и Google, CWV, что изменилось в 2025–2026 |
| [`docs/research/03-legal.md`](docs/research/03-legal.md) | Госреестры РФ, СПС, что сломано, юридические границы скрапинга |
| [`docs/research/04-accounting.md`](docs/research/04-accounting.md) | Бухгалтерия: ГИС МТ, ЭДО, 1С, налоги 2026 |
| [`docs/research/05-dev-stack.md`](docs/research/05-dev-stack.md) | Стек, локальные LLM, линтеры, профилирование, OCR и речь |
| [`docs/research/06-docs-images.md`](docs/research/06-docs-images.md) | PDF, DOCX, изображения, скрапинг, капчи, доступность из РФ |
| [`docs/CONTEXT.md`](docs/CONTEXT.md) | Состояние проекта, открытые вопросы, журнал работ |
| [`docs/DECISIONS.md`](docs/DECISIONS.md) | Архитектурные решения (ADR) |

## Что содержит отчёт

Только проверенные факты о внешних системах: HTTP-коды госсайтов, условия
доступа к API, цены, лицензии библиотек, версии, регуляторные изменения. Дата
проверки — 2026-10-01, способ проверки указан для каждого факта.

Утверждений о состоянии проектов отчёт не содержит: для них нужно читать код.
Список проверенных фактов о коде — в [`docs/CONTEXT.md`](docs/CONTEXT.md).

## Проверено и актуально на 2026-10-01

- **`kad.arbitr.ru/Kad/SearchInstances` → HTTP 451.** Не 403: вендор квалифицирует
  запрос как юридически недопустимый. Корень сайта отдаёт 200.
- **`bkr.sudrf.ru` — NXDOMAIN.** Хост удалён. `sudrf.ru` и `*.perm.sudrf.ru`
  работают, отдают windows-1251.
- **`egrul.nalog.ru/api.php` — 404.** ФНС переписала протокол: теперь
  `POST /` → токен → поллинг `search-result/{t}` → `vyp-download/{t}`.
- **У Яндекса нет Core Web Vitals.** Раздел об индексе скорости закомментирован
  в исходнике справки. Google использует CWV в ранжировании.
- **FAQ-разметка убрана из выдачи Google 07.05.2026.** `llms.txt` объявлен
  ненужным.
- **PyMuPDF — AGPL-3.0.** Для закрытого коммерческого продукта это риск.
- **`uv 0.12.18` исправил path traversal на Windows** (GHSA-2cv4-cqwr-gwf7). Актуально как общий факт: к коду пользователя не относится.
- **Санкционных блокировок инфраструктуры разработки из РФ нет.** npm, PyPI,
  GitHub, Hugging Face, jsdelivr, Docker Hub доступны.

## Что не удалось проверить

Цены Serpstat/Similarweb/Rushstat/PixelTools/КлючиСО · сроки отчётности и КБК ·
практика претензий 1С к интеграторам · Т-Банк и Сбер API · ФЗ № 228-ФЗ о пороге
УСН · скорость локальных моделей на конкретном железе. Полный список — в
[`docs/CONTEXT.md`](docs/CONTEXT.md).
## Быстрый старт

```bash
git clone https://github.com/AlexanderKuzikov/Observer.git
cd Observer
```

Репозиторий не содержит кода — только документация. Следующий шаг по
проработке зафиксирован в [`docs/CONTEXT.md`](docs/CONTEXT.md).

## Статус

**v0.1** — исследование завершено, ожидает выборочной проработки по частям.

## Лицензия

[Apache-2.0](LICENSE) © Alexander Kuzikov

