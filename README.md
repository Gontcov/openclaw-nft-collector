# NFT Intelligence Collector (MVP)

Сборщик теперь формирует **два отчёта**:

- `premint-report.json` — pre-mint анализ по X/Twitter и mint-метаданным.
- `daily-nft-report.json` — daily market анализ по OpenSea и shortlist под Nansen deep-dive.

Полезные файлы:

- `docs/SETUP_STEP_BY_STEP.md` — пошаговая настройка с нуля.
- `docs/NANSEN_CREDITS.md` — справка по кредитам Nansen.
- `docs/PREMINT_REPORT_SCHEMA.md` — схема `premint-report.json`.
- `docs/DAILY_REPORT_SCHEMA.md` — схема `daily-nft-report.json`.
- `docs/DISCOVERY_RULES.md` — правила discovery и decision labels.
- `docs/BUDGET_GUARDRAILS.md` — лимиты и защита бюджета.
- `docs/FEDYA_PREMINT_PROMPT.md` — готовый prompt для ручного pre-mint разбора.
- `docs/FEDYA_POSTMINT_PROMPT.md` — готовый prompt для post-mint и daily report разбора.

## Источники MVP

- **X API**: аккаунт проекта, твиты, базовая social-динамика, seed discovery.
- **OpenSea API**: floor, volume, holders, events, свежие market candidates.
- **Nansen**: пока только для финальных кандидатов после shortlist.

Если источник не настроен или отдал ошибку, сборщик:

- пишет `null`, где это допустимо,
- фиксирует предупреждение в `meta.warnings`,
- не выдумывает отсутствующие данные.

## Быстрый старт

```bash
cd /Users/nastyagontcova/Desktop/OpenClaw/nft-collector
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python src/collect_daily_report.py
```

Этот запуск создаёт оба файла:

- `premint-report.json`
- `daily-nft-report.json`

## Что заполнять в конфиге

- `config/candidates.json` — manual market candidates для OpenSea.
- `config/premint_watchlist.json` — premint проекты, которые ты хочешь точно отслеживать.
- `config/premint_seeds.json` — seed-аккаунты и search queries для controlled auto-discovery.

## Ключевые переменные `.env`

- `OUTPUT_PATH`
- `PREMINT_OUTPUT_PATH`
- `OPENSEA_API_KEY`
- `X_API_BEARER_TOKEN`
- `PREMINT_WATCHLIST_PATH`
- `PREMINT_SEEDS_PATH`
- `DAILY_SHORTLIST_LIMIT`

Nansen пока оставляем как deep-dive слой для финалистов:

- `NANSEN_API_KEY`
- `NANSEN_COLLECTION_ENDPOINT_TEMPLATE`

## Запуск по расписанию на VPS

Под пользователем `openclaw`:

```cron
10 8 * * * /home/openclaw/nft-collector/.venv/bin/python /home/openclaw/nft-collector/src/collect_daily_report.py >> /home/openclaw/nft-collector/collector.log 2>&1
```

## Интеграция с OpenClaw

Храни оба отчёта в доступном для OpenClaw месте, например:

```env
OUTPUT_PATH=/home/openclaw/data/daily-nft-report.json
PREMINT_OUTPUT_PATH=/home/openclaw/data/premint-report.json
```

Тогда Федя сможет:

- ежедневно читать `daily-nft-report.json`,
- отдельно анализировать `premint-report.json` для решения `mint / watch / skip`.
