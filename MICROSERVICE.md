# Microservice Design

## Frameworks and Libraries

- FastAPI
- SQLAlchemy
- Decouple
- Black
- Ruff
- Pytest

**Why FastAPI?**: Is a modern, high-performance web framework for building APIs.\
  Benchmarks show it's three times faster than Django and fifteen times faster than Flask, making it perfect for high-performance.\
  FastAPI's **dependency injection system** is one of its most powerful features, allowing you to manage dependencies in a clean, modular, and testable way .

**Why SQLAlcemy?** SQLAlchemy is a powerful and flexible Object-Relational Mapping (ORM) library for Python, and here are the key reasons to use it in your FastAPI project.
**Why use decouple?** Python Decouple is a library that helps manage configuration settings by separating them from your code.

**Why use python black?** Python Black is a code formatter that automatically formats your code to a consistent style. 

**Why use ruff?** Python Ruff is a fast, modern linter and formatter for Python.

**Why use pytest?** Pytest is a popular testing framework for Python, known for its simplicity, flexibility, and powerful features.

**Why I'm not using Hexagonal architecture, use-cases, DDD, etc?** Bacause it looks like a simple CRUD API and over-engineering introduces layers, abstractions, and patterns that makes the code harder to read, debug, and maintain.
 
## Project Structure Overview
```
app/
│── api/                # Routes (Views/Health)
│── core/               # App Configs
│── models/             # Models Pydantic / ORM
│── repositories/       # External data access
│── services/           # Businees rules
│── utils/              # Exceptions and validators
├── main.py             # Main app
├── requirements.txt
└── .env.sample
```

## Architecture Overview
api/: Defines API routes and endpoints.\
core/: Centralizes configuration and initialization.\
models/: Defines data structures and validation rules.\
repositories/: Handles data access, caching, queues and database operations.\
services/: Contains business logic and orchestrates operations.\
utils/: Provides utility functions and custom exceptions.