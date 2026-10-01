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

## Ключевые находки

- **`kad.arbitr.ru` отдаёт HTTP 451** — DDoS-Guard детектит headless на уровне wasm.
- **`bkr.sudrf.ru` — NXDOMAIN**, хост удалён. `bsr.sudrf.ru` не отвечает с российского IP.
- **`egrul.nalog.ru/api.php` — 404**, ФНС переписала протокол целиком.
- **PyMuPDF — AGPL-3.0**: риск для закрытого коммерческого продукта.
- **У Яндекса нет Core Web Vitals** — скорость влияет через поведенческие факторы.
- **FAQ rich result убран из Google** 7 мая 2026. `llms.txt` Google объявил ненужным.
- **TypeScript 7** (Go-порт) — сборки в 7,7–11,9× быстрее. **`tsup` мёртв** → `tsdown`.
- **Спецификация MCP 2026-07-28** — breaking change, stateless core.

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