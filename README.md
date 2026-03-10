# devOps_lab6_docker_compose

## Цель работы
Освоить создание многосервисных приложений с использованием **Docker Compose**, реализовать:
- веб-приложение на Flask с использованием Redis для хранения счётчика посещений
- систему мониторинга на базе Prometheus + Grafana
- внешний мониторинг доступности HTTP-эндпоинтов через Blackbox Exporter
- нативный экспорт собственных метрик приложения в Prometheus
- визуализацию ключевых показателей в Grafana

## Подготовка среды и сетевая настройка
* Работа выполнялась на виртуальной машине с **Ubuntu** (VirtualBox).
* Тип сети: **NAT**
* Для доступа к сервисам с хост-машины настроен **проброс портов** в VirtualBox:
  - 8000 → 8000 (Flask-приложение)
  - 3000 → 3000 (Grafana UI)
* На гостевой машине установлен Docker и необходимые плагины:
  ```bash
  sudo apt update
  sudo apt install docker.io docker-compose-v2
   ```

## Создание структуры проекта

В домашней директории создана следующая структура файлов и каталогов:

    
    devOps_lab6_docker_compose/
    ├── compose/ 
    │   ├── app.py
    │   ├── Dockerfile
    │   ├── requirements.txt
    │   └── compose.yaml
    └── prom/  
      ├── compose.yaml
      ├── prometheus/
      │   └── prometheus.yml
      └── grafana/
          └── datasource.yaml
    
    
## Разработка
**Flask + Redis:**
  * **app.py** — веб-приложение со счётчиком посещений на Redis
  * **Dockerfile** — сборка образа на базе python 3.10-alpine
  * **compose.yaml** — два сервиса: web (Flask) и redis (образ redis:alpine)

**Prometheus + Grafana:**
  * **compose.yaml** — сервисы prometheus, grafana, blackbox
  * **prometheus.yml** — конфигурация сбора метрик (сам Prometheus, Blackbox, Flask)
  * **datasource.yaml** — автоматическое подключение Prometheus в Grafana

## Запуск
**Flask + Redis:**

     
      cd compose
      docker compose up -d --build
     
**Prometheus + Grafana:**

      cd ..
      cd prom
      docker compose up -d
    
## Проверка результатов  

**Проверяем счетчик:**

<img width="653" height="137" alt="image" src="https://github.com/user-attachments/assets/3d0b2b18-127f-4e38-ba2d-65e48f7931c4" />

**Заходим на http://localhost:8080**

<img width="305" height="91" alt="image" src="https://github.com/user-attachments/assets/ec56a983-e26c-4712-821e-7374390227ae" />

**Захохим в Grafana UI http://localhost:3000**

<img width="634" height="332" alt="image" src="https://github.com/user-attachments/assets/445054df-1ea3-4d47-a89e-dc7e2a4b92f0" />

Панель **HTTP Probe Overview** показывает статус проверки трёх целей:
- http://10.0.2.15:8000 — **UP**, код 200, время ответа ~0.01 с;
- https://etis.psu.ru — **DOWN** (т.к. доступен только в университетской сети);
- https://student.psu.ru — **UP**, код 200, TLS 1.3, время ответа ~1.45 с.

<img width="999" height="240" alt="image" src="https://github.com/user-attachments/assets/5b8b1509-78a1-4023-9eaa-add6b8e09ee4" />

Панель **HTTP Probe Duration** показывает, сколько секунд занимал каждый запрос:
- https://etis.psu.ru — длительные таймауты (до 4.5–5 с), затем падение до 0 (таймаут);
- http://10.0.2.15:8000 и https://student.psu.ru — стабильное низкое время ответа.
    

**Создаем новый dashboard и указываем следущие настройки:**

<img width="485" height="299" alt="image" src="https://github.com/user-attachments/assets/4d85b6b3-b62c-4ec0-8d69-cfa5aa043906" />

**Визуализация количества посещений:**

<img width="685" height="348" alt="image" src="https://github.com/user-attachments/assets/3327ed63-73b5-4bc9-a3a1-b3723971ae27" />
