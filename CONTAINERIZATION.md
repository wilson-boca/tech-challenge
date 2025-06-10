# Containerization

To run the complete application, we need to run the API and the consumer.
You can find [here](#dockerfile-for-full-application) the docker-compose file that will also run the database, redis and rabbitmq.\
The dev images are using the **debugpy** to allow debugging inside the continer.\
Once we are using multistage building, the target production remove the debug and uses custom start CMD.

To create a docker container with the API run:
```shell
docker build -t api:latest --file app/Dockerfile app
```

## Dockerfile for the API container
```dockerfile
FROM python:3.11-slim AS builder

WORKDIR /app

# Install dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements to create a layer with the dependencies only
COPY requirements.txt .

# Create and activate virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy all code
COPY . .

# Expose port
EXPOSE 5590 5678

# Run cmd
CMD ["python3", "-Xfrozen_modules=off", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", "app.main:application", "--reload", "--host", "0.0.0.0", "--port", "5590"] 


FROM python:3.11-slim AS production

WORKDIR /app

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy application code
COPY --from=builder /app /app

# Expose prd port
EXPOSE 80

# Run prd cmd
CMD ["uvicorn", "app.main:application", "--host", "0.0.0.0", "--port", "80"]
```

This is the Dockerfile for the RabbitMQ consumer.
The consumer is using graceful shutdown, so it will fininish gracefully when the container is stopped.
```shell
docker build -t consumer:latest --file consumer/Dockerfile consumer
```

# Dockerfile for RabbitMQ consumer
```dockerfile
FROM python:3.11-slim AS builder

WORKDIR /consumer

# Install dependencies of the system
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create and activate virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy source
COPY . .

# Expose ports
EXPOSE 5678

CMD ["python", "-Xfrozen_modules=off", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "consumer/main.py"]

FROM python:3.11-slim AS production

WORKDIR /consumer

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy application code
COPY --from=builder /consumer .

# Run cmd
CMD ["python", "consumer/main.py"]
```

## Dockerfile for full application
```yaml
services:
  api:
    build:
      context: ./app
    restart: unless-stopped
    depends_on:
      - postgres
      - redis
      - rabbitmq
    ports:
      - "5590:5590"
      - "5678:5678"
    volumes:
      - ./app:/app
    networks:
      - backend
    environment:
      - POSTGRES_URI=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/db
      - RABBITMQ_URI=amqp://${RABBITMQ_USER}:${RABBITMQ_PASSWORD}@rabbitmq:${RABBITMQ_PORT}
      - REDIS_URI=redis://${REDIS_USER}:${REDIS_PASSWORD}@redis:${REDIS_PORT}
  worker:
    build:
      context: ./consumer
    restart: unless-stopped
    depends_on:
      - postgres
      - rabbitmq
    ports:
      - "5679:5679"
    volumes:
      - ./consumer:/app
    networks:
      - backend
    environment:
      - POSTGRES_URI=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/db
      - RABBITMQ_URI=amqp://${RABBITMQ_USER}:${RABBITMQ_PASSWORD}@rabbitmq:${RABBITMQ_PORT}
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      PGPORT: ${POSTGRES_PORT}
    ports:
      - "${POSTGRES_PORT}:${POSTGRES_PORT}"
    expose:
      - "${POSTGRES_PORT}"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "${REDIS_PORT}:${REDIS_PORT}"
    volumes:
      - redis_data:/data
    networks:
      - backend
  rabbitmq:
    image: rabbitmq:3-management-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=${RABBITMQ_USER}
      - RABBITMQ_DEFAULT_PASS=${RABBITMQ_PASSWORD}
    ports:
      - "${RABBITMQ_PORT}:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - backend
networks:
  backend:
    name: app-backend
volumes:
  postgres_data:
    name: postgres-data
  redis_data:
    name: redis-data
  rabbitmq_data:
    name: rabbitmq-data
```

To run the application, you can use the following command:
```shell
docker-compose build
docker-compose up -d
```
