ARG PYTHON_VERSION=3.10-slim

FROM python:${PYTHON_VERSION}

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Instalar dependencias esenciales para compilar
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /code
WORKDIR /code

# Instalar Poetry
RUN pip install --no-cache-dir poetry

# Copiar archivos de dependencias
COPY pyproject.toml poetry.lock* /code/

# CAMBIO CLAVE: Forzar modo no-paquete dinámicamente y limpiar la caché de Fly
RUN poetry config virtualenvs.create false \
    && poetry config package-mode false \
    && poetry install --no-interaction --no-ansi

# Copiar el código de la app
COPY . /code

ENV SECRET_KEY "AFwkWPL8AuNDcNrBLAG5vEwSzhsBzrBfRCfrWrVj7sVrpRrKkn"

# Recolectar estáticos sin romper si falta la base de datos temporalmente
RUN python manage.py collectstatic --noinput || true

EXPOSE 8000

CMD ["daphne","-b","0.0.0.0","-p","8000","dispatcharr.asgi"]
