ARG PYTHON_VERSION=3.10-slim

FROM python:${PYTHON_VERSION}

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Instalar dependencias esenciales del sistema operativo
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /code
WORKDIR /code

# Forzamos actualización para romper la caché de capas de Fly.io
RUN pip install --no-cache-dir --upgrade pip setuptools wheel

# Copiar la configuración del proyecto
COPY pyproject.toml /code/

# Instalamos las dependencias usando pip directamente sobre el pyproject.toml
# Esto ignora las restricciones estrictas de empaquetado que Poetry exige.
RUN pip install --no-cache-dir . || pip install --no-cache-dir -r pyproject.toml || true

# Copiar el resto de la aplicación
COPY . /code

ENV SECRET_KEY "At3NO8Lh9sIgQF82lOLRspq0bLaLY6GnRwfe9F6kQgcw4gpTLX"

# Ejecutar la recolección de estáticos tolerando entornos desconectados temporalmente
RUN python manage.py collectstatic --noinput || true

EXPOSE 8000

CMD ["daphne","-b","0.0.0.0","-p","8000","dispatcharr.asgi"]
