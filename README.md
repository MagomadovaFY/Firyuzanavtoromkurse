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

```
## 🖥️ Сервер (server.py)

import grpc
import time
import random
import logging

# Импортируем сгенерированные из .proto файла классы
import metrics_pb2
import metrics_pb2_grpc

# Настройка логирования
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def generate_metrics():
    """
    Генератор, который создает поток метрик для отправки на сервер.
    """
    value = 45.0
    
    for i in range(5):
        value += random.uniform(-1, 1) + 2.0
        value = max(0, value)
        
        metric = metrics_pb2.Metric(
            name="cpu_usage",
            value=value,
            timestamp=int(time.time()),
            tags={"source": "mac_client", "iteration": str(i)}
        )
        
        logging.info(f"Отправка метрики: {metric.name} = {metric.value:.2f}%")
        yield metric
        time.sleep(1)

def run():
    """Основная функция клиента"""
    channel = grpc.insecure_channel('localhost:50051')
    stub = metrics_pb2_grpc.MetricsCollectorStub(channel)
    
    logging.info("=" * 50)
    logging.info("Клиент запущен. Подключение к серверу localhost:50051...")
    logging.info("Начинаю отправку потока метрик")
    logging.info("=" * 50)
    
    try:
        response = stub.CollectMetrics(generate_metrics(), timeout=10)
        
        logging.info("=" * 50)
        logging.info("Получен ответ от сервера:")
        logging.info(f"Всего метрик: {response.total_count}")
        logging.info(f"Сумма значений: {response.total_sum:.2f}")
        logging.info(f"Сообщение: {response.message}")
        logging.info("=" * 50)
        
    except grpc.RpcError as e:
        logging.error(f"Ошибка при вызове RPC: {e.code()} - {e.details()}")
    except Exception as e:
        logging.error(f"Непредвиденная ошибка: {e}")

if __name__ == '__main__':
    run()
