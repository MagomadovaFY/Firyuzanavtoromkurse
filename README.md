# 📘 Лабораторная работа №1

## Вариант 11

### Тема: gRPC-сервис MetricsCollector (Client Streaming RPC)

**Студент:** Магомадова Фирюза  
**Дата выполнения:** 6 марта 2026

---

## 📌 Задание

Реализовать gRPC-сервис **MetricsCollector** с методом **CollectMetrics**.

Метод должен принимать **поток метрик** от клиентского приложения (**Client streaming RPC**) и возвращать статистику обработки.

---

## 🎯 Цель работы

- Освоить принципы удаленного вызова процедур (RPC)
- Изучить фреймворк gRPC и язык определения интерфейсов Protocol Buffers
- Реализовать клиент-серверное приложение на Python
- Получить практические навыки работы с Client streaming RPC

---

## 🧩 Описание сервиса

| Параметр | Значение |
|----------|---------|
| **Название сервиса** | MetricsCollector |
| **Метод** | CollectMetrics |
| **Тип RPC** | Client streaming RPC |
| **Входные данные** | Поток метрик (Metric) |
| **Выходные данные** | Статистика обработки (MetricsSummary) |


---

## 📄 Контракт сервиса (metrics.proto)

```protobuf
syntax = "proto3";

package metrics;

// Сервис для сбора метрик
service MetricsCollector {
    // Client streaming RPC: клиент отправляет поток метрик,
    // сервер возвращает один ответ с итоговой статистикой
    rpc CollectMetrics(stream Metric) returns (MetricsSummary) {}
}

// Сообщение, представляющее одну метрику
message Metric {
    string name = 1;               // Имя метрики (например, "cpu_usage")
    double value = 2;              // Значение метрики
    int64 timestamp = 3;           // Временная метка (Unix timestamp)
    map<string, string> tags = 4;  // Теги для дополнительной фильтрации
}

// Сообщение-ответ с итоговой статистикой
message MetricsSummary {
    int32 total_count = 1;    // Общее количество полученных метрик
    double total_sum = 2;      // Сумма всех значений метрик
    string message = 3;        // Сообщение о статусе обработки
}


