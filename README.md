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
python -m grpc_tools.protoc -I proto --python_out=. --grpc_python_out=. proto/metrics.proto
metrics_pb2.py
metrics_pb2_grpc.py
class MetricsCollectorServicer(metrics_pb2_grpc.MetricsCollectorServicer):
    def __init__(self):
        self.metrics = []
        
    def CollectMetrics(self, request_iterator, context):
        count = 0
        logging.info("Начало приема метрик")
        
        for metric in request_iterator:
            self.metrics.append({
                'name': metric.name,
                'value': metric.value,
                'unit': metric.unit,
                'source': metric.source,
                'time': metric.timestamp
            })
            count += 1
            logging.info(f"Получено: {metric.name} = {metric.value}")
            
        return metrics_pb2.CollectMetricsResponse(
            received_count=count,
            status="OK"
        )
<img width="705" height="30" alt="image" src="https://github.com/user-attachments/assets/93a1610c-34b6-42e8-aa7b-68aec599bd5f" />
class MetricsGenerator:
    def __init__(self):
        self.hostname = socket.gethostname()
        
    def get_cpu(self):
        return {
            'name': 'cpu_usage',
            'value': random.uniform(10, 90),
            'unit': '%',
            'source': self.hostname,
            'timestamp': int(time.time())
        }
    
    def get_memory(self):
        return {
            'name': 'memory_usage',
            'value': random.uniform(20, 80),
            'unit': '%',
            'source': self.hostname,
            'timestamp': int(time.time())
        }
def make_metrics():
    for i in range(5):
        if i % 2 == 0:
            data = gen.get_cpu()
        else:
            data = gen.get_memory()
            
        msg = metrics_pb2.Metric()
        # заполнение полей...
        print(f"  {i+1}. {msg.name} = {msg.value:.1f}")
        yield msg
        time.sleep(1)

response = stub.CollectMetrics(make_metrics())
python server.py
2026-03-06 13:49:12 - Сервер запущен на порту 50051
<img width="585" height="140" alt="image" src="https://github.com/user-attachments/assets/964d0679-1639-431c-a3a1-b58edf7c1d06" />

<img width="844" height="261" alt="image" src="https://github.com/user-attachments/assets/bd998980-9d1c-4a5d-b3c9-112e4a82b78a" />
