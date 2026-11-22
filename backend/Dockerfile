# backend/Dockerfile
FROM python:3.13-slim

WORKDIR /app

# install Python deps
COPY backend/requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir --index-url https://pypi.org/simple -r /tmp/requirements.txt

# copy backend code
COPY backend /app

# run as non-root user (least privilege)
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser \
    && mkdir -p /app/logs && chown -R appuser:appgroup /app

USER appuser

EXPOSE 8000
CMD ["sh", "-c", "python manage.py migrate --noinput && python manage.py runserver 0.0.0.0:8000"]

