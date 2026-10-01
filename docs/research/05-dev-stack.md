# Часть 5. Стек, локальные LLM, инструменты разработки

Проверено 2026-10-01 по GitHub releases, npm/PyPI registry и официальным docs.

**Границы:** часть описывает внешние инструменты. Рекомендации по внедрению,
привязанные к проектам пользователя, удалены — они строились на догадках о его
коде. Что реально применяется в проектах, проверено отдельно (см. `CONTEXT.md`).

## 1. Локальные LLM

### 1.1 Модели и железо

| Модель | Тип | Квант | Вес | Актуальность |
|---|---|---|---|---|
| Qwen3.6-27B | dense 27B | Q4_K_M | ~12,6 GiB | Apache-2.0 |
| **Qwen3.6-35B-A3B** | MoE 35B / 3B active | Q4_K_M | ~16,3 GiB | 16.04.2026. 256 экспертов, Gated DeltaNet, Apache-2.0 |
| Qwen3.6-35B-A3B | то же | Q5_K_M | ~20,4 GiB | — |
| gpt-oss-20b | MoE 21B / 3,6B active | MXFP4 нативный | ~12 ГБ | Apache-2.0 |
| Qwen3-8B-class / Phi-4-mini | dense | Q4/Q5 | 5–6 ГБ | фолбэк |

**MoE vs dense:** у MoE меньше *активных* параметров (быстрее compute на токен),
но **веса всё равно все 35B** — столько же RAM/VRAM и то же время загрузки с
диска по PCIe. При частичном offload на PCIe-bound сценарии преимущество
испаряется. Старые бэкенды читают конфиг MoE, но генерируют неверно.

Источники: `huggingface.co/Qwen/Qwen3.6-35B-A3B`, GGUF-конверсии
`bartowski/Qwen_Qwen3.6-35B-A3B-GGUF`.

❓ Скорости на конкретном железе **не измерены**. Оценки по классу (10–18 t/s
для 35B-A3B Q4) требуют проверки через `llama-bench`.

### 1.2 Рантаймы

| Рантайм | Лицензия | Актуальность | Примечание |
|---|---|---|---|
| **llama.cpp** | MIT | **b11312 (01.10.2026), релиз daily** | CPU+CUDA+Vulkan, спекулятивный декодинг |
| **Ollama** | MIT | v0.35.0 (28.09.2026) | OpenAI-совместимый API на :11434 |
| **LM Studio** | бесплатно для работы с 08.07.2025 | 0.4.x | GUI + локальный API |
| vLLM | Apache-2.0 | 0.21 | Linux + CUDA, tens-of-GPU |
| SGLang | Apache-2.0 | активный | дата-центры |

### 1.3 Спекулятивный декодинг

llama.cpp поддерживает **11 реализаций** через `--spec-type`:

| Тип | Механика | Применимость |
|---|---|---|
| `draft-simple` | отдельная draft-модель (`-md`) | есть EAGLE3-чеки для Qwen3 |
| `draft-eagle3` | 1-слойный трансформер читает hidden states целевой | **выше acceptance, чем standalone draft** |
| `draft-mtp` | MTP-головы самой модели | зависит от модели |
| `ngram-simple` / `ngram-map-k` / `ngram-map-k4v` / `ngram-mod` | паттерны в собственной истории токенов, без модели | рефакторинг кода, повторяющиеся структуры |
| `ngram-cache` | внешняя статистика | спецслучай |
| `draft-dflash` / `draft-dspark` | block-diffusion | только Qwen3-таргеты |

```bash
--spec-default                      # включает ngram-mod
--spec-type ngram-mod,ngram-map-k4v
-md <file.gguf>                     # draft-модель
--spec-draft-ngl auto|all|N
--backend-sampling
```

Бенчмарк: `tools/server/bench/speed-bench`. Спекулятивный декодинг
математически не меняет выход модели.

Готовые EAGLE-3 чеки: `AngelSlim/Qwen3-a3B_eagle3`,
`RedHatAI/gpt-oss-20b-speculator.eagle3`.

### 1.4 KV-кэш

Актуально явное управление типами: `-ctk/-ctv` (`f32`, `f16`, `bf16`, `q8_0`,
`q4_0`, `q4_1`, `iq4_nl`, `q5_0`, `q5_1`). Для 4K контекста на Qwen3.6 KV-кэш
в f16 упирается в VRAM — понижение K-типа освобождает десятки мегабайт без
заметной потери.

Не мешать три цифры: теоретический вес, размер GGUF, рантайм-потребление
(веса + KV + буферы + mmproj + overhead). Замерять логом старта и
`nvidia-smi`.

### 1.5 Подводный камень для агентов

`chat-template`: разные версии Qwen используют разные шаблоны. Признаки поломки —
литералы `system`/`user` в выводе, повторы, невалидный JSON в tool call. Не лечится
удалением спец-токенов.

## 2. События года в инструментах

### 2.1 TypeScript 7 — Go-порт

**TypeScript 7.0 вышел 08.07.2026.** Нативный Go-порт компилятора и language
service (проект tsgo). Официальные бенчмарки Microsoft: полные сборки **в
7,7–11,9 раза быстрее** TypeScript 6. npm `latest` = **7.0.2**. TypeScript 6.0 —
финальный релиз из JS-кода.

### 2.2 Rolldown и бандлеры

| Событие | Дата |
|---|---|
| **Rolldown 1.0 stable** — API зафиксирован, semver, можно пинить `^1.0.0`. В 10–30× быстрее Rollup | **07.05.2026** |
| **Vite 8.0 stable** — Rolldown дефолтный бандлер, заменил и esbuild, и Rollup. Rust-стек | **12.03.2026** |
| Next.js 16 — Turbopack stable и дефолтен для `next dev` и `next build` | 2025–2026 |
| **tsup — maintenance mode**, мейнтейнеры рекомендуют `tsdown` | с 08.2025 |

`tsdown` — Rolldown + Oxc, полная совместимость с опциями tsup, команда
`migrate`, AI-скиллы для агентов.

### 2.3 uv — дыра на Windows

**`uv 0.12.18` исправил path traversal при установке wheel на Windows**
(GHSA-2cv4-cqwr-gwf7). Актуальная версия — **0.12.21**.

Применимость: дыра срабатывает только при установке wheel через `uv`. В
окружении пользователя `uv.lock` нет ни в одном проекте, `uv tool list` пуст —
версию в PATH даёт копия внутри `AppData\Local\hermes\bin`. К его проектам
отношения не имеет.

### 2.4 Прочие версии

| Инструмент | Актуальная версия |
|---|---|
| ruff | 0.16.9 (24.09.2026) |
| golangci-lint | v2.14 (24.09.2026), модульный путь `/v2` |
| Psalm | 6.19.1 (29.09.2026) + 7.0.0-beta22 |
| Zod | 4.6.5 (25.09.2026), v4 переписан, ~14× быстрее |
| tRPC | 11.19.0 (16.09.2026) |
| Deno | 2.9.7 (16.09.2026) |
| Bun | 1.4.2 — 1.4 переписан с Zig на Rust |
| pnpm | 12.8.2 (30.09.2026) |
| Playwright | 1.63.0 |
| k6 | 2.3.0 (21.09.2026), есть MCP-сервер |
| OpenTelemetry Collector | v0.162.0 (29.09.2026), graduated из CNCF, semconv 1.44.0 |
| axe-core | 4.13.0 |
| web-vitals | v6.0.0 (21.07.2026), Soft Navigation |
| sitespeed.io | v40.0 (05.05.2026) |

## 3. Статический анализ

| Инструмент | Языки | Лицензия |
|---|---|---|
| golangci-lint v2 | Go | MIT |
| Ruff | Python | MIT, Rust-бинарь, 900+ правил |
| PHPStan | PHP | MIT, уровень 10 |
| Psalm | PHP | MIT. Понимает PHP 8.5: `#[\Override]`, asymmetric visibility. В 7.0 taint-анализ стал детерминированным |
| Semgrep | 30+ | OSS-движок бесплатно; Pro ~$30–40/разработчик/мес |
| ty (Astral) | Python | MIT, preview |

⚠️ Psalm 7 в beta больше года. Стабильная ветка — 6.x.

## 4. Нагрузка и профилирование

| Инструмент | Статус |
|---|---|
| **k6** | Go-бинарь, AGPL-3.0. `--scenario`, `--once`, MCP-сервер (`k6 x mcp`) |
| Locust | 2.45 (09.07.2026), установка через `uvx` |
| Artillery | жив, уступает k6 по скорости |
| **Vegeta** | **последний релиз 31.10.2025 — застыл** |
| **samply** | Единственный профайлер с off-CPU стеками на Windows (ETW). UI — Firefox Profiler |
| **py-spy** | Attach к живому Python-процессу, оверхед ~нулевой |
| pprof | встроен в Go |

Разделять микро-бенчмарк (регрессии в CI) и нагрузочный тест (SLO на ожидаемом
трафике).

## 5. Снапшот-тесты

| Инструмент | Статус |
|---|---|
| **Playwright `toHaveScreenshot`** | без внешнего сервиса. С 1.60 (05.2026) — официальный MCP-сервер и **ARIA snapshots** |
| **Playwright Test Agents** | planner / generator / **healer** — самопочинка тестов |
| Chromatic | зрелое, платное |
| Percy | **приобретена BrowserStack** |
| BackstopJS | Playwright ушёл далеко вперёд |

ARIA snapshots — качественный скачок: агент читает семантическое дерево, а не
только скриншот.

## 6. Веб-производительность

| Инструмент | Статус |
|---|---|
| **sitespeed.io** | v40.0, Coach-правила под 2026 |
| Lighthouse CI | рабочий |
| WebPageTest API | нужен API-ключ от Catchpoint, ~100 тестов/сутки на публичном инстансе |
| axe-core | 4.13.0, WCAG 2.0/2.1/2.2 |
| pa11y | жив |

### Спецификация

- **INP заменил FID**, порог ≤200 мс. В 2026 INP растёт, LCP — нет.
- **web-vitals v6.0.0** — Soft Navigation; сборки `standard` и `attribution`
  (attribution даёт атрибуцию виновника в INP).
- **Speculation Rules API** — prefetch/prerender. **Chromium-only**, для **MPA**.
  Начинать с prefetch: prerender расходует ресурс на то, что может не открыться.
- **View Transitions API** — cross-document для MPA, opt-in через CSS.

## 7. OCR — русский язык как ключевой критерий

| Инструмент | Версия | CPU | Русский |
|---|---|---|---|
| **RapidOCR** | 3.9.2 | **да, ONNX** | ru-конфиги |
| **PaddleOCR** | 3.7.0 | да | **отдельная ru-модель в v5_multi_languages**. В PP-OCRv6 русский не заявлен |
| **PaddleOCR-VL-1.6** | 0,9B, **96,3% OmniDocBench** | GPU желателен | 109 языков, русский явно |
| **Tesseract** | **5.5.3** | да, всегда | `rus.traineddata`, **среднее на кривых сканах** |
| Surya | 0.22.1 | слабо, GPU | слабо |
| Docling (IBM) | **2.131.0** | да | да |
| marker | 2.0.0 — **полная поддержка CPU с 2.0** | да | через Surya-OCR 2 |
| MinerU | 4.0.10 | да, `basic` = ONNX, 2 ГБ RAM | ancient/rare chars |
| EasyOCR | 1.7.2 — **2024-09** | да | ru-веса |

**Честно о русском:**

- **Печатный скан 2000-х:** PaddleOCR PP-OCRv5 (ru) **≥** Tesseract 5.5.3.
- **1950-е, машинопись, дореформенная орфография: готового решения нет.**
  Дообучение требует отдельного проекта с размеченным датасетом.
- **Таблицы:** MinerU > Docling > marker. Для **рамчатых** таблиц **Camelot 2.0
  lattice** быстрее и надёжнее нейросетей.

## 8. Речь

| Инструмент | Русский | Примечание |
|---|---|---|
| **faster-whisper** | хороший на large-v3 | **на CUDA быстрее whisper.cpp в 5–7×** |
| whisper.cpp | приемлемо | на Windows с **Vulkan — 10–12× быстрее** faster-whisper на CPU. **На NVIDIA наоборот** |
| **large-v3** | приемлемо, лучшее | 1,55B, MIT |
| **large-v3-turbo** | **хуже на русском** | дистилляция под английский |
| **WhisperX** | — | **диаризация + выравнивание** |
| Vosk | есть | качество заметно ниже |
| Parakeet TDT | **нет русского** | не подходит |

Для русского: **large-v3 (не turbo) + faster-whisper на CUDA**, квант int8.

## 9. Агенты и CLI

| Агент | Локальные модели | Примечание |
|---|---|---|
| **opencode** | да, first-party с Ollama | рабочий инструмент |
| **Crush** (Charm) | да | v0.50.1 (17.03.2026) |
| **Qwen Code** | да | форк Gemini CLI от Alibaba |
| Goose (Block) | да | «расширения» вместо MCP |
| aider | да | **устаревает** |
| Cline / Roo | да | VS Code-расширения, не CLI |

## 10. MCP вне браузера

См. `01-mcp.md` для спецификации и рисков. Здесь — только применимость.

| Сервер | Что даёт |
|---|---|
| **github/github-mcp-server** | PR/issues/CI, remote-версия без локального Docker |
| **Serena** | LSP-семантика: `find_symbol`, `replace_symbol_body`. 40+ языков |
| **k6 MCP** | агент сам пишет нагрузочные тесты |

## Не верифицировано

Скорости локальных моделей на конкретном железе (нужен `llama-bench`) ·
acceptance rate EAGLE-3 draft (нужен `speed-bench`) · Semgrep Pro точная цена ·
Doguhan/Phrase exact-цены WebPageTest API · MCP-сервер Ollama/LM Studio и
grep.app — отсутствие не доказано.

