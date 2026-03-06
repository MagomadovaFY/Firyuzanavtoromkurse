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

+-------------+ gRPC +-------------+
| Клиент | <---------------> | Сервер |
| client.py | (stream метрик) | server.py |
+-------------+ порт +-------------+
| 50051 |
| |
генерирует сохраняет
метрики метрики
(random) в список
| |
v v
cpu_usage, metrics[]
memory_usage в памяти


### Компоненты системы

* **`client.py`**  
  Генерирует тестовые метрики (CPU, память) и отправляет их потоком на сервер.

* **`server.py`**  
  Принимает поток метрик, логирует каждую полученную метрику и сохраняет их в памяти.

* **`proto/metrics.proto`**  
  Контракт взаимодействия между клиентом и сервером. Описывает методы сервиса и структуры сообщений.

* **`metrics_storage`**  
  Хранилище метрик в оперативной памяти (список словарей).

---

## Описание реализации

В рамках лабораторной работы был создан gRPC-сервис **MetricsCollector**.

Описание сервиса выполнено в файле:

Сервис содержит один метод:

* **`CollectMetrics`** — принимает поток метрик от клиента.

Параметр **`stream`** в запросе означает, что клиент отправляет несколько сообщений, а сервер получает их последовательно и возвращает один ответ после завершения потока.

📷 **Скриншот:** содержимое файла **proto/metrics.proto**
<img width="622" height="442" alt="image" src="https://github.com/user-attachments/assets/519f8255-f388-4d94-af13-41749e43eaf7" />

---

## Генерация gRPC-кода

После создания **metrics.proto** была выполнена команда:

```bash
python -m grpc_tools.protoc -I proto --python_out=. --grpc_python_out=. proto/metrics.proto

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
