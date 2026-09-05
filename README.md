# Платформа персональных рекомендаций TechnoMart

Архитектурное проектирование AI-сервиса рекомендаций для ритейлера TechnoMart:
C4-модель в LikeC4 и спецификации API в OpenAPI 3.1. Учебный проект курса
OTUS «AI Architect» (ДЗ 05).

## Онлайн-версия

Интерактивные диаграммы (LikeC4), прямые ссылки на каждое представление:

- [Диаграмма системного контекста (C1)](https://sablev.github.io/otus-ai-architect-hw05/view/index)
- [Диаграмма контейнеров (C2)](https://sablev.github.io/otus-ai-architect-hw05/view/platform--containers)
- [Компоненты Recommendation Serving API (C3)](https://sablev.github.io/otus-ai-architect-hw05/view/platform--recommendation-serving-api--components)
- [Компоненты Description Generation Worker (C3)](https://sablev.github.io/otus-ai-architect-hw05/view/platform--description-generation-worker--components)
- [Сценарий «Пользователь запрашивает рекомендацию» (диаграмма последовательности)](https://sablev.github.io/otus-ai-architect-hw05/view/platform--request-recommendation--dynamic)
- [Сценарий офлайн-предгенерации описаний (диаграмма последовательности)](https://sablev.github.io/otus-ai-architect-hw05/view/platform--pregenerate-descriptions--dynamic)

Спецификации API в SwaggerUI:

- **Recommendation Serving API**: <https://sablev.github.io/otus-ai-architect-hw05/api/recommendation-serving-api/>
- **Recommendation Backend API**: <https://sablev.github.io/otus-ai-architect-hw05/api/recommendation-backend/>

## Состав репозитория

| Путь | Содержимое |
|---|---|
| `c4/` | Модель LikeC4: системный контекст (C1), контейнеры (C2), компоненты (C3) и два сценария-диаграммы последовательности |
| `c4/exports/` | PNG-экспорты всех представлений — диаграммы видны прямо в веб-интерфейсе GitHub без запуска инструментов |
| `api/recommendation-serving-api/openapi.yaml` | Внутренний контракт AI-сервиса: вызывается Recommendation Backend по `X-Api-Key` |
| `api/recommendation-backend/openapi.yaml` | Публичная граница для браузерного виджета: сессия через cookie, без секрета |

## Локальный запуск

Все инструменты упакованы в Docker-образы, на хосте нужен только Docker.
Команды выполняются из корня репозитория.

**Интерактивные диаграммы** (dev-сервер LikeC4 на http://localhost:5173):

```bash
docker compose -f c4/docker-compose.yaml up -d
```

**Просмотр спецификаций и моки** (SwaggerUI на http://localhost:8080 и
http://localhost:8081, моки Prism на http://localhost:4010 и
http://localhost:4011):

```bash
docker compose -f api/docker-compose.yaml up -d
```

Мок Serving API сразу отвечает примером из спецификации:

```bash
curl -H "X-Api-Key: local-dev" \
  "http://localhost:4010/v1/recommendations?slot=product_card_cross_sell&product_id=SKU-100234"
```

## Ключевые архитектурные решения

- **Живого вызова LLM в онлайн-сценарии нет.** Тексты подборок
  предгенерированы офлайн-воркером и лежат в кэше; на критическом пути
  ответа (бюджет 200 мс) — только чтения по ключу.
- **AI Service разделён** на Recommendation Serving API (онлайн-выдача) и
  Description Generation Worker (офлайн-генерация): несовместимые профили
  нагрузки и отказа.
- **Деградация прозрачна для виджета:** при отказе Serving API Backend
  отдаёт резервный rule-based блок из локального кэша, фоново
  синхронизируемого с сайтом.

## Публикация

Сайт на GitHub Pages собирается workflow
[.github/workflows/pages.yml](.github/workflows/pages.yml): `likec4 build`
для диаграмм плюс статические страницы SwaggerUI для обеих спецификаций.
