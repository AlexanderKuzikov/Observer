# Часть 6. PDF, DOCX, изображения, скрапинг, капчи, доступность из РФ

Проверено 2026-10-01 через PyPI JSON API, npm registry, GitHub API и живые
HTTP-проверки с российского IP (Москва, AS8402).

## 1. PDF

| Инструмент | Версия | Лицензия | Качество/скорость |
|---|---|---|---|
| **PyMuPDF** | 1.28.2 (06.08.2026) | **AGPL-3.0** ⚠️ | Самая быстрая экстракция текста. 1.28.2: `find_tables()` получил `use_layout/union/refine`, `Document.apply_css()`, поддержка архивов |
| **PyMuPDF4LLM** | 1.28.2 | **AGPL-3.0** ⚠️ | PDF → markdown для LLM, быстро на том же MuPDF |
| pypdf | 6.19.0 (16.09.2026) | MIT-ish | Медленнее PyMuPDF в 3–5× на больших файлах, без бинарных зависимостей |
| **pdfplumber** | 0.11.10 (15.06.2026) | MIT | Лучший для анализа координат и таблиц с рамками |
| **pikepdf** | 10.16.0 (29.09.2026) | MPL-2.0 | Лучший для структурных правок и **вложений** (`.attachments` — канонический API) |
| pypdfium2 | 5.13.0 (13.08.2026) | BSD | Рендер быстрее Ghostscript. С 17.0.0 ocrmypdf предпочитает его |
| pdfminer.six | 20260107 | MIT | LTChar/LTTextBox, основа Camelot 1.x. Развитие замедлилось |
| **Camelot** | 2.0.0 (04.06.2026) | MIT | **Смена бэкенда: pypdf+pdfminer → playa-pdf.** Умеет borderless и скан-таблицы |
| qpdf | 12.4.2 (27.09.2026) | Apache-2.0 | CLI-нормализация/валидация. Обязателен для склейки/починки |
| GhostPDL | push 30.09.2026 | AGPL-3.0 | Нужен только для PDF/A и как fallback-растеризатор ocrmypdf |
| **ocrmypdf** | 17.13.0 (28.09.2026) | MPL-2.0 | OCR-слой, **`--deskew`, `--rotate-pages`** |
| deskew | 1.6.1 (03.06.2026) | MIT | Чистый deskew по Hough, без OCR, быстро |
| borb | 3.0.9 | MIT | Альтернатива ReportLab, медленнее fpdf2 |

### Rust/Go альтернативы

| Инструмент | Состояние | Лицензия |
|---|---|---|
| **lopdf** (Rust) | 0.45.0 (08.09.2026), 21,9 млн загрузок | MIT |
| printpdf (Rust) | 0.12.8, 3,1 млн | MIT |
| ravif (Rust, AVIF) | 0.13.0, 49,5 млн | Apache/MIT |
| **oxipdf** (Rust) | **хобби, заброс**, 301 загрузка | — |
| pdf-rs / pdfium-rs | заброшены | MIT |
| mupdf-rs | форк messense, полузаброшен | AGPL-3.0 ⚠️ |
| UniPDF (Go) | v5.1.0, живой | **коммерческая** |
| signintech/gopdf (Go) | v0.22.2 | Apache-2.0 |
| **go-pdf/fpdf** | **архивирован** (03.2025) | MIT |
| ledongthuc/pdf (Go) | живой, извлечение текста | BSD-3 |

### Вывод по PDF

PyMuPDF — ядро, всё остальное — точечно. Для коммерческой продажи **AGPL-3.0
критичен**: закрытый продукт без платы Artifex нарушает copyleft. Безопасная
связка: pikepdf + pdfplumber + pypdf.

### DESKEW — три рабочих пути в 2026

1. **`ocrmypdf --deskew` + `--rotate-pages`** — Leptonica, самое точное
   (учитывает текстовые блоки). Требует рендера страницы.
2. **`deskew 1.6.1`** — чистый Hough, без OCR, быстро, по одному изображению.
3. **OpenCV `minAreaRect` / `HoughLinesP`** — если opencv уже в проекте
   (`opencv-python 5.0.0.93`).

> **В PyMuPDF встроенного deskew нет** — только `page.set_rotation()` и
> `page.derotation_matrix`, то есть поворот всего, а не выравнивание по тексту.

## 2. OCR и layout

| Инструмент | Версия | CPU | Русский | URL |
|---|---|---|---|---|
| Tesseract | **5.5.3** (24.07.2026) | да | `rus.traineddata` есть, **среднее на кривых сканах** | github.com/tesseract-ocr/tesseract |
| PaddleOCR | 3.7.0 (11.06.2026) | да (ONNX/OpenVINO: medium 1,40 с на Xeon, tiny 0,20 с) | **отдельная ru-модель в v5_multi_languages** | github.com/PaddlePaddle/PaddleOCR |
| **RapidOCR** | 3.9.2 (21.07.2026) | **да, ONNX — основной режим** | ru-конфиги | github.com/RapidAI/RapidOCR |
| docTR | 1.1.0 (21.08.2026) | да | через multilingual backbones | github.com/mindee/doctr |
| Surya | 0.22.1 | слабо, GPU | слабо | github.com/VikParuchuri/surya |
| marker | 2.0.0 (20.07.2026) — **полная поддержка CPU с 2.0** | да | через Surya-OCR 2 | github.com/datalab-to/marker |
| **Docling** | 2.131.0 (29.09.2026) | да | да | github.com/docling-project/docling |
| Unstructured | 0.27.10 | да | да | github.com/Unstructured-IO/unstructured |
| **MinerU** | 4.0.10 (29.09.2026) | да: `basic` = ONNX, 2 ГБ RAM | да, отдельный пайплайн под ancient/rare chars | github.com/opendatalab/MinerU |
| EasyOCR | 1.7.2 — **2024-09, полумёртв** | да | ru-веса есть | github.com/JaidedAI/EasyOCR |

**marker 2.0 — важное изменение:** три режима. `balanced` (76,0% olmOCR-bench,
GPU-ориентирован), `fast` (rf-detr/onnx layout + pdftext, 66,6%),
`--disable_ocr` (только текстовый слой, **полностью на CPU**, 23,7 стр/с).

**MinerU лицензия:** Apache-2.0, **но** с дополнительными условиями — при MAU >
100 млн или выручке > $20 млн/мес требуется отдельная коммерческая лицензия, при
online-сервисе обязателен атрибут «используется MinerU». **Авто-прекращение
лицензии** при нарушении.

### Честно про русский язык

- **Печатный скан 2000-х:** PaddleOCR PP-OCRv5 (ru) ≥ Tesseract 5.5.3 с
  tessdata_best. Paddle лучше на кириллице, особенно на грязных сканах.
- **1950-е, машинопись, дореформенная орфография: ничего готового нет.** Ни
  PaddleOCR, ни Tesseract не обучены на дореформенной орфографии/машинописи.
  Единственный путь — дообучение (`tesseract -l rus` с новым traineddata либо
  PaddleOCR fine-tune), что означает отдельный проект с размеченным датасетом.
- `tesseract.js` 7.0.0 + `@tesseract.js-data/rus` 1.0.0 (2023) — работает в
  браузере, но rus-датасет с 2023 не обновлялся.
- **Таблицы/формулы:** MinerU (PaddleOCR-VL-1.6, 96,33% OmniDocBench) > Docling >
  marker. Для **рамчатых** таблиц — **Camelot 2.0 lattice** быстрее и надёжнее
  нейросетей.

## 3. DOCX/OOXML

| Инструмент | Версия | Лицензия | Состояние |
|---|---|---|---|
| python-docx | 1.2.0 (16.06.2025) | MIT | Жив (push 08.2026), релизов мало |
| **docxtpl** | 0.20.2 (13.11.2025) | LGPL-2.1 | Жив — единственный живой OSS-вариант с нормальными лицензиями |
| docxcompose | 2.2.0 (02.06.2026) | MIT | Жив |
| **python-pptx** | 1.0.2 — **2024-08** | MIT | **мёртв**, 540 открытых issue |
| **openpyxl** | 3.1.5 — **2024-06** | MIT | **мёртв** |
| xlsxwriter | 3.2.9 | BSD | умеренно жив |
| **LibreOffice** | 26.8.0 fresh / 26.2.6 still | MPL-2.0 | Основной headless-конвертер |
| **ONLYOFFICE DocumentServer** | 9.4.0 (19.05.2026) | **AGPL-3.0** | Self-hosted + REST API, бесплатно |
| **Typst** | **0.15.1 (17.07.2026)** | Apache-2.0 | **Самый быстрый генератор PDF с таблицами**, нативный русский шрифт |
| Pandoc | 3.12 (29.09.2026) | GPL-2.0 | Жив. docx reference-doc |
| **docx (npm)** | 9.8.1 (28.09.2026) | MIT | Жив |
| docxtemplater | 21.09.2026 | коммерческий freemium | Жив — нужен для сложных циклов/условий в таблицах |
| reportlab / fpdf2 | 5.0.1 / 2.8.9 | MIT | Живы |
| aspose-words / GroupDocs | 26.9.0 | платно | — |
| Spire.Doc | 14.9.1 (29.09.2026) | платно + Free с лимитами | — |
| **docx2pdf** | 0.1.8 — **2021** | MIT | **мёртв**, требует установленный Word |
| **unoconv** | — | — | **мёртв**, заменён на unoserver |

### Ответ по ключевым вопросам

- **Генерация с нуля (текст + таблицы, русский):** для DOCX — python-docx +
  docxtpl. **Для PDF-вывода — Typst 0.15.1 выигрывает радикально**:
  компиляция миллисекунды против десятков секунд LibreOffice, таблицы
  первоклассные, шрифты подставляются локально, zero-dependency на сервере.
  Pandoc — для конверсии существующего markdown/HTML, не для генерации.
- **Шаблоны (подстановка в существующий .docx):** docxtpl — единственный живой
  OSS-вариант. Платная альтернатива docxtemplater нужна только для сложных
  циклов/условных блоков в таблицах.
- **DOCX → PDF без Word:** LibreOffice headless (`soffice --headless
  --convert-to pdf`). ONLYOFFICE — альтернатива с REST API и лучшей
  MS-совместимостью, но AGPL.
- **python-pptx и openpyxl мертвы** — xlsx писать через xlsxwriter.

## 4. Изображения

| Инструмент | Версия | Скорость/качество | Заметка |
|---|---|---|---|
| **sharp** | 0.35.5 (27.09.2026) | Лидер для Node. Билдит libvips 1.3.4 (libvips 8.18.7) | jxl/webp/avif/heif/jp2/gif/tiff/raw |
| wasm-vips | 0.0.19 (29.09.2026) | Fallback без натива | github.com/wasm-vips/wasm-vips |
| **libvips** | 8.18.7 (26.09.2026) | Основа sharp | `vipsthumbnail` для CLI-батчинга |
| ImageMagick | 7.1.2-32 (27.09.2026) | Универсален, **медленнее libvips в 2–5× на ресайзе** | Apache-2.0 |
| **libavif** | 1.4.2 (26.05.2026) | `avifenc` — лучший AVIF-энкодер | BSD-3 |
| libjxl | 0.12.0 | **Внедрение в браузеры остановилось** — нет в Chrome/Safari/Firefox | BSD-3 |
| Pillow | 12.3.0 | Медленнее libvips, без внешних зависимостей | HPND |
| pillow-avif-plugin | 1.6.0 (22.07.2026) | AVIF/HEIC для Pillow | BSD-2 |
| imagecodecs | 2026.8.16 | Максимум форматов | BSD-3 |
| opencv-python | 5.0.0.93 | Для CV | Apache-2.0 |
| **@napi-rs/image** | 1.15.0 (17.09.2026) | Быстрая альтернатива sharp на чистом Rust | MIT |
| **imagemin** | 9.0.1 (07.03.2025) | **Половина плагинов мертва** (imagemin-webp 8.0.0 = 2023, mozjpeg 10.0.0 = 2021) | не трогать |
| @jsquash/* | webp 1.5.0, avif 2.1.1, resize 2.1.1 | Только браузер, не Node-сервер | WASM |
| **squoosh / @squoosh/lib** | 0.0.0 (2021) / 0.5.3 (2023) | **мёртвы** | — |
| **FLIF** | последний коммит 2015 | **мёртв** | — |

### Ресайз без потери качества

- Только **downscale** — никогда не увеличивать без явной необходимости.
- `sharp.kernel` — **`lanczos3`** по умолчанию, лучший выбор; `mitchell` для
  чуть меньшего ореола.
- **shrink-on-load**: libvips декодирует JPEG сразу в 1/2, 1/4, 1/8 — кратно
  быстрее без потери качества. Уже включено в sharp.
- Предварительное уменьшение (`thumbnail`) перед ресайзом большого кадра убирает
  алиасинг.
- `sharp.rotate()` по EXIF-ориентации (`.withMetadata()` для сохранения EXIF),
  `failOn: 'none'`.
- Прогрессивный JPEG (`progressive: true`) + `mozjpeg: true` — меньше вес.
- ICC-профиль: `withIccProfile()` — обязательно для печатных задач.

**Для батча:** `sharp` в Node, `vipsthumbnail` для файловой обработки без Node,
Pillow + pillow-avif-plugin в Python.

## 5. Скрапинг и антибот

| Инструмент | Последний релиз | Лицензия | Статус |
|---|---|---|---|
| Scrapy | 2.19.0 (10.09.2026) | BSD-3 | **жив** |
| httpx | 0.28.1 — **2024-12** | BSD | замедлился, но рабочий |
| **curl_cffi** | **0.16.3 (02.09.2026)** | MIT | **жив** — TLS impersonation через curl-impersonate |
| curl-impersonate (низ) | v0.6.1 (03.2024) | MIT | заброшен, но используется повсеместно |
| **tls-client** | 1.0.1 — **2024-02** | — | **мёртв** |
| nodriver | 0.50.3 (13.05.2026) | **AGPL-3.0** ⚠️ | жив, но AGPL |
| Camoufox | 0.5.6 (06.09.2026) | MPL-2.0 | README: **«under development, may not be suitable for stable production use»** |
| **patchright** | **1.63.0 (20.09.2026)** | Apache-2.0 | **жив** — патчит Playwright, скрывает CDP-утечки |
| **undetected-chromedriver** | 3.5.5 — **2024-02** | GPL-3.0 | **мёртв фактически.** 2,5 года без релиза |
| rebrowser-puppeteer / playwright | 24.8.1 / 1.52.0 — **2025-05** | MIT | **мёртвы**, заменены на patchright |
| puppeteer-extra (+stealth) | 3.3.6 / 2.11.2 — **2023-03** | MIT | **мёртв** |
| **Scrapling** | 0.4.15 (23.08.2026) | BSD-3 | **жив, 84,8k★** — самый быстрорастущий anti-bot SDK |
| DrissionPage | 4.1.1.4 (27.05.2026) | BSD | жив — управление реальным Chrome |
| Playwright / Puppeteer | 1.63.0 / 25.12.0 | Apache-2.0 | живы |
| got / undici / axios | 16.0.0 / 8.11.2 / 1.20.0 | MIT | живы |
| @sparticuz/chromium | 153.0.0 (11.09.2026) | — | жив, для Lambda/контейнеров |

### Что реально работает против российских госсайтов

**sudrf.ru (ГАС «Правосудие»):** главная `index.php?id=300` отдаётся **без
капчи**, на старом PHP/jQuery. Капча — на `bsr.sudrf.ru` (банк судебных актов).
С российского IP `bsr.sudrf.ru` **не отвечает вообще** (TCP timeout, DNS
84.42.111.136 резолвится) — то ли фильтрация по репутации IP, то ли сам хост.
**Нужен российский/домашний IP или резидентный прокси.**

**kad.arbitr.ru:** сервер **ddos-guard** (видно по `Server: ddos-guard` и кукам
`__ddg1_/8_/9_/10_`). Капча `pravocaptcha` появляется **не сразу, а после
превышения числа попыток**: текст «Превышено количество попыток ввода кода» +
«До сброса ~3 минуты». То есть первые N запросов проходят без капчи, дальше нужен
решатель. **Не капча на каждый запрос, а rate-limit с последующей блокировкой на
3 минуты.**

### Практический вывод

1. **TLS:** `curl_cffi` с `impersonate="chrome"` — базовая линия, дёшево, работает
   против ddos-guard.
2. **Браузер:** `patchright` (живой, Apache-2.0) — вторая линия. `Camoufox` —
   только если детект сложный, с пониманием риска «не для прода».
3. **Фингерпринт IP решает всё.** ddos-guard и sudrf фильтруют по репутации IP.
   Домашний IP / мобильный прокси важнее любого стелса.
4. `undetected-chromedriver` — не использовать. 2 года без релиза.
5. tlsfuzzer — инструмент для **тестирования** TLS, не для обхода.

## 6. Капчи

### RuCaptcha — цены проверены 01.10.2026

| Тип | Цена за 1000 | Скорость |
|---|---|---|
| Капча-картинка | **18–44 ₽** | 2 сек |
| reCAPTCHA v2 | 65–160 ₽ | 3 сек |
| reCAPTCHA v3 / Enterprise | 99–160 ₽ | 2 сек |
| Cloudflare Turnstile | 99 ₽ | 2 сек |
| GeeTest | 160 ₽ | 4 сек |
| Arkose/FunCaptcha | 99–3000 ₽ | 18 сек |
| DataDome | 99 ₽ | 15 сек |

Цены в рублях, есть «Русская капча» как отдельный тип, оплата российскими
методами. Клиент `python-rucaptcha 6.8.0` (11.07.2026) — живой. Старый пакет
`rucaptcha 0.0.2` (2022) — мёртв.

### Альтернативы

| Сервис | Цена за 1000 | Комментарий |
|---|---|---|
| **RuCaptcha** | **18–44 ₽** | Лучшие рубли, русский тип капчи |
| anti-captcha.com | $0,50–0,95 | Старейший (2007), русский интерфейс, но **цены в USD, оплата картой РФ — риск** |
| 2captcha.com | £0,45–0,89 (картинка) | Выводит в **фунтах**. Оплата РФ-картой проблемна |
| **capTCHA.so** | — | **домен припаркован / Access Denied** — сервис фактически не работает |

### Локальное распознавание — честно

**Tesseract/EasyOCR/PaddleOCR общие модели капчи не решают.** `ddddocr` 1.6.1
(11.03.2026, MIT, 14,8k★) обучен именно на капчах, но на **китайских/простых
текстовых**. Умеет `classification` и `detection`, но **не решает Яндекс
SmartCaptcha / KeyCaptcha / CheckCaptcha** — там защита не в сложности картинки,
а в серверной валидации сессии и поведенческих паттернах.

Локальный браузерный распознаватель (PaddleOCR-ONNX tiny ~15 МБ, Tesseract.js)
технически реализуемо, но проигрывает экономически: добавляет 1–3 с на капчу,
которая на RuCaptcha стоит 0,02–0,04 ₽.

**Вывод:** для sudrf/kad/ФНС нужен платный сервис. RuCaptcha дёшев и окупается за
секунды. Локальный OCR имеет смысл только для массовой простой captcha-картинки.

## 7. Доступность из РФ — фактические замеры

Замеры 01.10.2026, Москва, AS8402 Vimpelcom.

| Ресурс | HTTP | Время TLS | Вердикт |
|---|---|---|---|
| registry.npmjs.org | 200 | **6,7 с** | работает, TLS-рукопожатие медленное |
| registry.npmmirror.com | 200 | 6,0 с | работает. **Ускорения против официального не даёт** (14,3 с полный запрос) |
| pypi.org / files.pythonhosted.org | 200 | 4,9 с | работает |
| mirrors.cloud.tencent.com/pypi | 200 | 6,0 с | **работает** — реально используемая альтернатива |
| pypi.tuna.tsinghua.edu.cn | **000** | — | **не работает с РФ** |
| mirrors.aliyun.com/pypi | 000/301 | — | не работает (301 на корень) |
| huggingface.co | 200 | 5,2 с | **работает!** модели PaddleOCR/Docling/marker качаются |
| hf-mirror.com | 200 | 12,1 с | работает, медленнее оригинала |
| modelscope.cn | 302 | — | отвечает (redirect), полноценная альтернатива HF |
| cdn.jsdelivr.net / unpkg / esm.sh | 200 | ~5,9 с | работают |
| github.com | 200 | 5,8 с | работает |
| codeload.github.com | 200 | 10,6 с (**DNS 5,2 с!**) | работает, DNS-резолв заметно медленный |
| fonts.googleapis.com | 200 | ~5,9 с | работает |
| fonts.gstatic.com | 404/000 | — | **нестабильно (2/3 успеха). Шрифты с Google Fonts рискованно тянуть в прод** |
| registry-1.docker.io | 401 (норма) | — | работает |
| ghcr.io | 000/403 | — | **проблемы — не использовать** |
| goproxy.cn | 200 | — | работает для Go |
| sum.golang.org | **000** | — | **недоступен** — Go-сборки требуют `GOSUMDB=off` |

### Итог

**В 2026 году санкционных блокировок на уровне инфраструктуры разработки НЕТ.**
PyPI, npm, GitHub, Hugging Face, jsdelivr, Docker Hub — всё доступно.
Блокируются ghcr.io, TUNA-pypi, sum.golang.org, нестабилен fonts.gstatic.com.

> **Выбор инструментов не ограничен доступностью — ограничен лицензиями и
> качеством.** Платные зарубежные SaaS (OpenAI, AWS, Cloudflare, Vercel,
> DigitalOcean) с российских карт не принимают — но инструменты выше
> полностью локальны и этой проблемы не имеют. Единственное исключение —
> облачные OCR API (Mathpix, ocr.space, Azure Document Intelligence, Google
> Document AI).

## 8. Привязка к проектам

| Проект | Стек на 2026 | Менять |
|---|---|---|
| **PDFtoText** | PyMuPDF 1.28.2 + pikepdf. OCR-слой — ocrmypdf 17.13 (`--deskew`/`--rotate-pages`) + Tesseract 5.5.3 rus | **Лицензия:** если продаётся как закрытый продукт — AGPL не подходит, переходить на pikepdf + pdfplumber + pypdf |
| **DOCX-Builder** | Typst 0.15.1 для PDF + python-docx/docxtpl если нужен .docx. Конвертация: LibreOffice 26.8 headless или ONLYOFFICE 9.4 | Не трогать без лицензионной необходимости. Если нужна чистота — Typst + docxtpl полностью MIT/LGPL |
| **DOCX-Ream** | docxtpl + docxcompose 2.2.0 | — |
| **DocuDeskew** | ocrmypdf `--deskew`/`--rotate-pages` для PDF; deskew 1.6.1 или OpenCV для картинок | **В PyMuPDF встроенного deskew нет** |
| **DocuMind** | Tesseract 5.5.3 + tessdata_best (rus) для печатного; PaddleOCR 3.7 PP-OCRv5 ru для грязных; Docling 2.131 для layout. **Все три работают на CPU без GPU** | Для доков 195-х: ни один готовый инструмент не подойдёт — нужен собственный дообученный traineddata |
| **Image-Converter** | sharp 0.35.5 или Pillow 12.3 + pillow-avif-plugin. AVIF — libavif 1.4.2 | Убрать imagemin и squoosh. JXL не тащить |
| **Resizer** | `kernel: lanczos3`, shrink-on-load, progressive + mozjpeg, withIccProfile | — |
| **Webp-Catalog-Builder / flat-webp-*** | sharp для ресайза, `vipsthumbnail` для файлового батча | — |
| **SudRF-Parser / CourtHarvester** | curl_cffi 0.16.3 (`impersonate="chrome"`) как базовая линия + patchright 1.63 как эскалация + RuCaptcha (18–44 ₽/1000) + резидентный российский IP | **Убрать** undetected-chromedriver, rebrowser-*, puppeteer-extra, tls-client. Camoufox — только эксперимент |

### Три критичных предупреждения

1. **PyMuPDF — AGPL-3.0.** Для закрытого коммерческого продукта в продажу это
   юридический риск. Держать MIT-стек.
2. **undetected-chromedriver мёртв**, несмотря на 12,8k★.
3. **Camoufox README прямо говорит, что проект не для продакшена.** Не строить на
   нём коммерческое решение.