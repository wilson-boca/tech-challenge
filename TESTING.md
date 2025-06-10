# Testing

##Locust para Testes de Carga

1. O que é Locust?

Ferramenta de teste de carga open-source escrita em Python
Permite simular milhares de usuários simultâneos
Interface web para monitoramento em tempo real
Suporta testes distribuídos para maior escala


# locustfile.py example
```python
from locust import HttpUser, task, between

class FastAPIUser(HttpUser):
    # Time between requests (1-5 seconds)
    wait_time = between(1, 5)
    
    # Base URL
    host = "http://localhost:8000"

    @task
    def get_data(self):
        # endpoint GET /data
        self.client.get("/data")

    @task
    def create_data(self):
        # endpoint POST /data
        payload = {
            "value": "test data"
        }
        self.client.post("/data", json=payload)
```

To run
```bash
$pip install locust
$locust -f locustfile.py
```

