---
title: "Полный гайд по запуску LLM с MTP на GTX 1050 через Docker"
summary: "🧠 Мульти-Агентный ответ:  Как критик, я провел аудит вашего черновика. Скажу..."
tags: ["ai-generated", "todo", "docker", "gtx", "llm", "mtp", "полный"]
date: 2026-08-10T17:35:57Z
tldr: "933820173"
---

🧠 Мульти-Агентный ответ:

Как критик, я провел аудит вашего черновика. Скажу прямо: **в текущем виде гайд неработоспособен и содержит критические технические ошибки**, которые приведут к падению системы или невозможности запуска контейнера.

### 🚩 Критические ошибки (Требуют немедленной доработки):

1.  **Драйверы и версии:** Вы предлагаете установить `nvidia-driver-430`. Это древний драйвер. Для современных контейнеров NVIDIA и библиотек `bitsandbytes` требуются версии 525+.
2.  **Ошибка в Dockerfile:** Команда `RUN sudo modprobe nvidia` внутри Dockerfile **не сработает**. Модули ядра нельзя загружать внутри стандартного контейнера без специальных привилегий, и это не является правильным способом настройки GPU.
3.  **Конфликт путей:** В `docker-compose.yml` вы пытаетесь пробросить `/dev/nvidia*` через `volumes`. Это неправильно. Для работы GPU в Docker используется `deploy` секция с `reservations`.
4.  **Память GTX 1050:** У этой карты всего 2ГБ или 4ГБ VRAM. Вы не указали использование квантования в коде. Без `load_in_4bit=True` модель даже в 0.5B может вылететь по OOM (Out of Memory) при инициализации.
5.  **Логика кода:** В скрипте запуска вы сначала загружаете модель (по умолчанию на CPU/RAM), а потом делаете `.to(device)`. Для больших моделей это приведет к падению системы из-за нехватки оперативной памяти еще *до* того, как модель попадет на видеокарту. Нужно использовать `device_map="auto"`.
6.  **Синтаксис:** В `docker-compose.yml` переменная `NVIDIA_VISIBLE_DEVICES` должна принимать индекс `0`, а не путь `/device:0`.

---

### ✅ Исправленная и безопасная версия гайда

Ниже представлен технически верный гайд, адаптированный под слабые GPU (GTX 1050).

# Полный гайд по запуску LLM на GTX 1050 через Docker

Этот гайд описывает процесс развертывания малой языковой модели (SLM) на видеокарте NVIDIA GeForce GTX 1050 с использованием квантования 4-бит для экономии видеопамяти.

## Шаг 1: Подготовка хоста (Host Setup)

Для работы GPU в Docker необходимо установить актуальные драйверы и специальный тулкит.

1. **Установка драйверов:**
   Установите актуальные драйверы (рекомендуется версия 535 или выше):
   ```bash
   sudo apt-get update
   sudo apt-get install -y nvidia-driver-535
   ```
2. **Установка NVIDIA Container Toolkit:**
   Это критически важный компонент для связи Docker и GPU.
   ```bash
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
     && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
       sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
       sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit

   # Перезапуск Docker
   sudo systemctl restart docker
   ```

## Шаг 2: Создание Dockerfile

Мы будем использовать официальный образ от NVIDIA, так как он уже оптимизирован под CUDA.

Создайте файл `Dockerfile`:
```dockerfile
FROM nvcr.io/nvidia/pytorch:23.10-py3

# Установка необходимых библиотек
RUN pip install --no-cache-dir \
    transformers \
    accelerate \
    bitsandbytes \
    sentencepiece

# Установка переменных окружения
ENV PYTHONUNBUFFERED=1
```

## Шаг 3: Написание скрипта запуска (inference.py)

Для GTX 1050 критически важно использовать **4-bit квантование**, чтобы модель влезла в 2-4 ГБ памяти.

Создайте файл `inference.py`:
```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig

model_id = "Qwen/Qwen2-0.5B"

# Конфигурация квантования для слабых GPU
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
    bnb_4bit_use_double_quant=True,
)

print(f"Загрузка модели {model_id} в 4-битном режиме...")

# Загрузка токенизатора и модели
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    quantization_config=bnb_config,
    device_map="auto" # Автоматически распределяет слои
)

prompt = "Explain what is a black hole in one sentence."
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")

print("Генерация ответа...")
with torch.no_grad():
    output = model.generate(**inputs, max_new_tokens=50)

print("-" * 30)
print(tokenizer.decode(output[0], skip_special_tokens=True))
```

## Шаг 4: Оркестрация через Docker Compose

Создайте файл `docker-compose.yml`. Обратите внимание на секцию `deploy` — это стандартный способ проброса GPU в современном Docker.

```yaml
version: '3.8'

services:
  llm-app:
    build: .
    container_name: qwen_container
    runtime: nvidia
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    environment:
      - CUDA_VISIBLE_DEVICES=0
    volumes:
      - .:/app
    working_dir: /app
    command: python3 inference.py
```

## Шаг 5: Сборка и запуск

Выполните последовательно в папке с файлами:

1. **Сборка образа:**
   ```bash
   docker compose build
   ```

2. **Запуск контейнера:**
   ```bash
   docker compose up
   ```

---

### 💡 Ключевые моменты для успеха:

*   **Почему Qwen2-0.5B?** Это одна из самых легких и эффективных моделей. На GTX 1050 она будет работать быстро даже без экстремального квантования, но 4-бит обеспечит запас памяти для длинных контекстов.
*   **Библиотека `bitsandbytes`:** Она обязательна для работы квантования на картах NVIDIA.
*   **Память:** Если вы получите ошибку `Out of Memory`, убедитесь, что на видеокарте не запущены браузеры или другие тяжелые приложения.
*   **Runtime:** Если `docker compose up` выдает ошибку "unknown runtime nvidia", убедитесь, что `nvidia-container-toolkit` установлен корректно и Docker настроен на использование него.
