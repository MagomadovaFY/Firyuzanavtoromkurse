# Firyuzanavtoromkurse
# 📘 Лабораторная работа №1

## Вариант 11

### Тема: gRPC-сервис MetricsCollector (Client Streaming RPC)

**Студент:** Магомадова Фирюза  
**Дата выполнения:** 6 марта 2026

---

## 📌 Задание

Реализовать gRPC-сервис **MetricsCollector** с методом:

Метод должен принимать **поток метрик** от клиентского приложения
(**Client streaming RPC**) и возвращать статистику обработки.

---

## 🏗 Архитектура решения

В работе реализована классическая **клиент-серверная архитектура**.

### Компоненты системы

* **Client (`client.py`)**  
  Генерирует тестовые метрики (CPU, память) и отправляет их потоком на сервер.

* **Server (`server.py`)**  
  Принимает поток метрик, логирует каждую полученную метрику и сохраняет их в памяти.

* **`proto/metrics.proto`**  
  Контракт взаимодействия между клиентом и сервером. Описывает сервис и структуры сообщений.

* **Сгенерированные файлы**  
  `metrics_pb2.py` и `metrics_pb2_grpc.py` — созданы автоматически для работы с gRPC.

---

## 🧩 Описание контракта (metrics.proto)

В файле `proto/metrics.proto` описан сервис и структуры данных:

```protobuf
syntax = "proto3";

package metrics;

service MetricsCollector {
    rpc CollectMetrics(stream Metric) returns (CollectMetricsResponse);
}

message Metric {
    string name = 1;
    double value = 2;
    string unit = 3;
    string source = 4;
    int64 timestamp = 5;
    map<string, string> tags = 6;
}

message CollectMetricsResponse {
    int32 received_count = 1;
    string status = 2;
}
## ⚙ Генерация gRPC-кода

После создания `task.proto` была выполнена команда:

```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. task.proto
```

В результате были автоматически созданы файлы:

* `task_pb2.py`
* `task_pb2_grpc.py`

Эти файлы обеспечивают взаимодействие клиента и сервера через gRPC.


---

## 🖥 Реализация сервера

Серверная часть реализована в файле:

```
server.py
```

Создан класс `TaskManager`, наследующийся от `TaskManagerServicer`.

Реализованы методы:

* `GetTasksForUser` — возвращает задачи с использованием `yield`;
* `UpdateTaskStatus` — изменяет статус задачи в памяти (имитация базы данных).

Сервер запускается на порту:

```
50051
```

и ожидает подключения клиентов.

---

📷 **Скриншот:** запущенный сервер
<img width="1020" height="342" alt="image" src="https://github.com/user-attachments/assets/8eb753f7-f00c-47ee-8117-6ee8abe79057" />


---

## 💻 Реализация клиента

Клиентская часть реализована в файле:

```
client.py
```

Подключение выполняется через:

```python
grpc.insecure_channel('localhost:50051')
```

Клиент позволяет:

* получить список задач пользователя;
* изменить статус выполнения задачи.

Получение задач реализовано через цикл `for`, который обрабатывает поток ответов сервера.

---

📷 **Скриншот:** получение списка задач
<img width="707" height="477" alt="image" src="https://github.com/user-attachments/assets/7ed9d7f9-300a-4f37-b4ad-d943bdcf6ba4" />


📷 **Скриншот:** изменение статуса задачи
<img width="2050" height="874" alt="image" src="https://github.com/user-attachments/assets/ae1a9737-6c0a-49c4-ba7a-53dd1fcb82ba" />


---

## 🧠 Используемые технологии

* Python 3
* gRPC
* Protocol Buffers
* Virtual Environment (venv)

---

## 🚀 Запуск проекта

### 1️⃣ Активация виртуального окружения

```bash
source venv_lab01/bin/activate
```

### 2️⃣ Запуск сервера

```bash
python server.py
```

### 3️⃣ Запуск клиента

```bash
python client.py
```

---

## 📊 Результат работы

Сервис корректно:

* возвращает поток задач пользователя (Server Streaming);
* изменяет статус выполнения задачи;
* обрабатывает запросы клиента через gRPC.

---

## 📌 Вывод

В ходе выполнения лабораторной работы:

* реализован gRPC-сервис TaskManager;
* изучена технология Server Streaming RPC;
* реализовано взаимодействие клиента и сервера;
* выполнена генерация кода на основе Protocol Buffers;
* реализовано изменение состояния данных через отдельный RPC-метод.
