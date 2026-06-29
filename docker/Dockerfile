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

# Instalar dependencias del sistema y herramientas de compilación
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    netcat-openbsd \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Instalar manejador de dependencias y extraer requerimientos
RUN pip install --no-cache-dir --upgrade pip

# Copiar configuración de dependencias de la raíz
COPY pyproject.toml /app/

# Como el proyecto usa pyproject.toml moderno, instalamos las dependencias directo
RUN pip install --no-cache-dir . || pip install --no-cache-dir -r requirements.txt || true

# Copiar todo el código de la aplicación
COPY . /app

# Limpiar directorio frontend local y copiar el compilado de la etapa 1
RUN rm -rf /app/frontend
COPY --from=frontend-builder /app/frontend/dist /app/frontend/dist

# Dar permisos de ejecución a los scripts del contenedor
RUN for f in /app/docker/entrypoint*.sh; do \
    if [ -f "$f" ]; then \
    sed -i 's/\r$//' "$f" && chmod +x "$f"; \
    fi; \
    done

# Exponer el puerto por defecto de la app
EXPOSE 8000

# Usar el entrypoint por defecto del proyecto
ENTRYPOINT ["/app/docker/entrypoint.sh"]
