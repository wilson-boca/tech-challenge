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