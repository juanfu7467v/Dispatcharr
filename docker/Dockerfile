ARG PYTHON_VERSION=3.10-slim

FROM python:${PYTHON_VERSION}

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Instalar dependencias para compilar paquetes nativos de Python (como psycopg2)
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /code
WORKDIR /code

# Actualizar el gestor de paquetes e instalar Poetry
RUN pip install --no-cache-dir --upgrade pip poetry

# Copiar las listas de dependencias
COPY pyproject.toml poetry.lock* /code/

# CORRECCIÓN CLAVE: Configuramos Poetry para instalar globalmente en el sistema 
# e indicamos explícitamente que no busque la metadata de un paquete propio del proyecto.
RUN poetry config virtualenvs.create false
RUN poetry install --no-root --no-interaction

# Copiar el resto del código del repositorio
COPY . /code

ENV SECRET_KEY "mUkz7ImIitzC2NoI26DTOJfPYBSLV3ZI2kCFZodMUjhc3xEmik"

# Recolectar estáticos tolerando faltas iniciales
RUN python manage.py collectstatic --noinput || true

EXPOSE 8000

CMD ["daphne","-b","0.0.0.0","-p","8000","dispatcharr.asgi"]
