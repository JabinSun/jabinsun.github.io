```Dockerfile
FROM python:3.10-slim AS builder
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"
WORKDIR /app
COPY ./requirements.txt .
RUN python3 -m venv /app/venv && \
    /app/venv/bin/pip install \
    -i https://mirrors.aliyun.com/pypi/simple \
    --no-warn-script-location \
    --no-cache-dir \
    -r requirements.txt

FROM python:3.10-slim-buster
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/app/venv/bin:$PATH"
WORKDIR /app
COPY --from=builder /app/venv /app/venv
COPY . /app
EXPOSE 5000
CMD ["/app/venv/bin/gunicorn", "app:app", "--bind", "0.0.0.0:5000"]
```

```Dockerfile
# python:<version>-slim / python:<version>-slim-bullseye / python:<version>-slim-buster
# ${version} >> ${version}-slim > ${version}-slim-bullseye > ${version}-slim-buster
FROM python:3.10-slim
LABEL maintainer="xxx@gmail.com"
EXPOSE 8000
# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE=1
# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1
# Install pip requirements
COPY requirements.txt .
RUN python -m pip install --no-cache-dir --upgrade -r requirements.txt
WORKDIR /app
COPY . /app
# Creates a non-root user with an explicit UID and adds permission to access the /app folder
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser
CMD ["uvicorn", "shortener_app.main:app", "--host", "0.0.0.0"]
```