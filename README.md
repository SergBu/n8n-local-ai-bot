# Мой первый AI-бот — n8n + Ollama

Локальный чат-бот на **n8n**, **Ollama** и **Llama 3.2 (3B)**. Пользователь пишет сообщение в чат n8n, узел AI Agent отправляет его локальной языковой модели и возвращает ответ на русском языке. Для работы не нужен ключ OpenAI API.

## Архитектура

```text
When chat message received ──► AI Agent
                                  ▲
                                  │ AI language model
                          Ollama Chat Model
                            (llama3.2:3b)
```

**Что реализовано в экспортированном workflow:** Chat Trigger, AI Agent с системной инструкцией отвечать по-русски и Ollama Chat Model (`llama3.2:3b`). Память диалога, Telegram, инструменты и публичный чат в этой версии **не настроены**.

## Требования

- Docker Desktop (Windows/macOS) или Docker Engine + Compose (Linux)
- Достаточно оперативной памяти для запуска модели; ориентир — от 8 ГБ для комфортных первых экспериментов
- Интернет для первоначального скачивания Docker-образов и модели (примерно 2 ГБ для модели)

## Быстрый запуск

1. Скачайте или клонируйте репозиторий и откройте терминал в его корне.
2. Создайте файл настроек:

   **PowerShell:**
   ```powershell
   Copy-Item .env.example .env
   ```

   **Linux/macOS:**
   ```bash
   cp .env.example .env
   ```

3. Запустите контейнеры:
   ```bash
   docker compose up -d
   ```
4. Скачайте модель:
   ```bash
   docker compose exec ollama ollama pull llama3.2:3b
   ```
5. Откройте **http://localhost:5678** и создайте учётную запись администратора при первом запуске.
6. В редакторе n8n создайте workflow и выберите **Import from File** (в меню `…`); импортируйте `workflows/my-first-ai-bot.json`.
7. В узле **Ollama Chat Model** создайте credentials типа **Ollama** с **Base URL** `http://ollama:11434` (без API-ключа). Сохраните credentials и выберите модель `llama3.2:3b`.
8. Откройте встроенный чат workflow и задайте вопрос, например: «Объясни, что такое Docker».

> **Важно:** поле Model находится в узле **Ollama Chat Model**, а не в окне **Ollama account**. Если n8n предлагает «Connect a model» для встроенного Assistant, это отдельная функция: для данного workflow внешний провайдер не нужен.

## Проверка и диагностика

```bash
docker compose ps
docker compose exec ollama ollama list
docker compose logs --tail=50 ollama
docker compose logs --tail=50 n8n
```

При ошибке `ECONNREFUSED` проверьте, что оба сервиса запущены, и что Base URL в credentials равен `http://ollama:11434`. Для проверки связи из контейнера n8n:

```bash
docker compose exec n8n node -e "fetch('http://ollama:11434/api/tags').then(r => console.log(r.status)).catch(console.error)"
```

## Управление

```bash
docker compose stop        # остановить
docker compose start       # запустить снова
docker compose down        # удалить контейнеры и сеть, сохранить данные
docker compose up -d       # обновить/пересоздать после изменения настроек
```

**Не выполняйте `docker compose down -v`, если хотите сохранить данные n8n и модели Ollama.** Данные хранятся в именованных Docker volumes.

## Безопасность

- В репозитории нет секретных ключей и идентификаторов исходного экземпляра n8n; привязку к Ollama credentials нужно создать после импорта.
- Порт n8n привязан к `127.0.0.1`, поэтому конфигурация предназначена для **локального запуска**, а не для публичного VPS.
- Для размещения в интернете нужны домен, HTTPS, обратный прокси, защита админки и отдельная настройка публичного чат-интерфейса.
- Не публикуйте `.env`, данные Docker volumes, реальные credentials и пользовательские сообщения.

## Для портфолио

**Стек:** n8n, Ollama, Llama 3.2, Docker, Docker Compose.

**Результат:** локальный AI-чат-бот, который принимает сообщения через Chat Trigger и генерирует ответы с помощью AI Agent и модели Ollama без платных облачных API.

Добавьте в `screenshots/` изображение workflow и скриншот диалога, а также ссылку на репозиторий в резюме.

## Дальнейшие улучшения

Память диалога (Simple Memory), Telegram-интеграция, обработка ошибок и публичная демоверсия с HTTPS.
