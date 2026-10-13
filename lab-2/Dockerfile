FROM python:3.11-slim AS builder

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir --target=/app/deps -r requirements.txt


FROM python:3.11-slim

# Создаем пользователя без root
RUN useradd -u 10001 -m appuser

WORKDIR /app

COPY --from=builder /app/deps /usr/local/lib/python3.11/site-packages

COPY app/ /app/app/
COPY requirements.txt /app/

ARG LAB_LOGIN=unknown
ARG LAB_TOKEN=unknown

LABEL org.lab.login=$LAB_LOGIN
LABEL org.lab.token=$LAB_TOKEN

ENV ROCKET_SIZE=Small
ENV LAB_LOGIN=$LAB_LOGIN
ENV LAB_TOKEN=$LAB_TOKEN
ENV PORT=8000

RUN chown -R appuser:appuser /app

USER appuser

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health').read()"

EXPOSE 8000

CMD ["python", "app/app.py"]