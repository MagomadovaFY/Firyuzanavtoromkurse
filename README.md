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
"""
Серверная часть gRPC-сервиса MetricsCollector.
Реализует Client streaming RPC для сбора метрик.
"""

import grpc
from concurrent import futures
import logging

# Импортируем сгенерированные из .proto файла классы
import metrics_pb2
import metrics_pb2_grpc

# Настройка логирования
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)


class MetricsCollectorServicer(metrics_pb2_grpc.MetricsCollectorServicer):
    """
    Класс, реализующий логику сервиса MetricsCollector.
    Наследуется от сгенерированного класса MetricsCollectorServicer.
    """
    
    def CollectMetrics(self, request_iterator, context):
        """
        Реализация метода CollectMetrics (Client streaming RPC).
        
        Параметры:
            request_iterator: итератор объектов Metric от клиента
            context: объект контекста вызова
            
        Возвращает:
            MetricsSummary: объект с итоговой статистикой
        """
        logging.info("=" * 50)
        logging.info("Получен запрос на сбор метрик")
        
        # Инициализация счетчиков
        total_count = 0
        total_sum = 0.0
        
        # Обработка каждой метрики из потока
        for metric in request_iterator:
            total_count += 1
            total_sum += metric.value
            logging.info(f"Получена метрика: {metric.name} = {metric.value:.2f}")
        
        logging.info(f"Обработка завершена. Получено метрик: {total_count}, сумма: {total_sum:.2f}")
        logging.info("=" * 50)
        
        # Возвращаем ответ с итоговой статистикой
        return metrics_pb2.MetricsSummary(
            total_count=total_count,
            total_sum=total_sum,
            message=f"Успешно обработано {total_count} метрик"
        )


def serve():
    """Запуск gRPC сервера"""
    # Создаем сервер с пулом потоков
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    
    # Добавляем реализацию сервиса
    metrics_pb2_grpc.add_MetricsCollectorServicer_to_server(
        MetricsCollectorServicer(), server
    )
    
    # Настраиваем порт
    port = 50051
    server.add_insecure_port(f'[::]:{port}')
    server.start()
    logging.info(f"Сервер запущен на порту {port}")
    
    try:
        server.wait_for_termination()
    except KeyboardInterrupt:
        logging.info("Сервер остановлен")


if __name__ == '__main__':
    serve()
