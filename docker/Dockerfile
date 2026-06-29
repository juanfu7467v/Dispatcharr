# --- Etapa 1: Compilar el Frontend ---
FROM node:20-slim AS frontend-builder
WORKDIR /app/frontend
COPY ./frontend /app/frontend
RUN rm -rf node_modules || true && \
    npm install --no-audit --progress=false && \
    npm run build

# --- Etapa 2: Imagen Final de Producción ---
FROM python:3.10-slim AS final

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PORT=8000

# Instalar dependencias esenciales del sistema operativo
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    netcat-openbsd \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /code

# Actualizar pip e instalar la herramienta nativa de empaquetado del proyecto (uv)
RUN pip install --no-cache-dir --upgrade pip uv

# Copiar archivos de configuración de dependencias de la raíz
COPY pyproject.toml /code/

# Instalar los paquetes del proyecto usando uv directamente en el sistema global del contenedor
RUN uv pip install --system . || pip install --no-cache-dir . || true

# Copiar el código fuente completo del backend
COPY . /code

# Limpiar rastros viejos del frontend y meter los estáticos optimizados de la etapa 1
RUN rm -rf /code/frontend
COPY --from=frontend-builder /app/frontend/dist /code/frontend/dist

# Dar permisos correctos a scripts de ejecución por lote si existen
RUN for f in /code/docker/entrypoint*.sh; do \
    if [ -f "$f" ]; then \
    sed -i 's/\r$//' "$f" && chmod +x "$f"; \
    fi; \
    done

ENV SECRET_KEY "uspKxjGpyLMb6KYBj7NXgxL9VaM0ZBcZwsPSFMZtLo87xyHlv6"

# Generar archivos estáticos para Django
RUN python manage.py collectstatic --noinput || true

EXPOSE 8000

CMD ["daphne","-b","0.0.0.0","-p","8000","dispatcharr.asgi"]
