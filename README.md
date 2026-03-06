# Firyuzanavtoromkurse
# 📘 Лабораторная работа №1

## Вариант 11

### Тема: gRPC-сервис MetricsCollector (Client Streaming RPC)

**Студент:** Магомедова Фирюза  
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
python -m grpc_tools.protoc -I proto --python_out=. --grpc_python_out=. proto/metrics.proto
