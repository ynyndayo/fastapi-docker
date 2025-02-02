ARG BASE_IMAGE=python:3.8.9-slim

# ==========================
FROM ${BASE_IMAGE} as build
WORKDIR /usr/src/app
COPY pyproject.toml poetry.lock ./

RUN pip install poetry \
    && poetry export --without-hashes -f requirements.txt > requirements.txt \
    && pip install -r requirements.txt

# ==========================
FROM ${BASE_IMAGE} as prod
ENV PYTHONUNBUFFERED=1
WORKDIR /work/src

COPY --from=build /usr/local/lib/python3.8/site-packages /usr/local/lib/python3.8/site-packages

COPY /app .
CMD ["uvicorn", "app:app"]


# ==========================
FROM ${BASE_IMAGE} as dev

ARG USERNAME
ARG USER_UID
ARG USER_GID

WORKDIR /usr/src/app
# rootだとコンテナ外からファイル編集できなくなる

COPY pyproject.toml poetry.lock /usr/src/app/

RUN echo USER_ID=${USER_UID} && \
    apt-get update && \
    apt-get install -y git && \
    groupadd --gid ${USER_GID} ${USERNAME} && \
    useradd -s /bin/bash --uid ${USER_UID} -G sudo \
        --gid ${USER_GID} -m ${USERNAME} \
        -p $(python -c "import crypt, getpass, pwd; print(crypt.crypt('password','\$6\$salt\$'))") && \
    pip install poetry && \
    # 仮想環境を作らない
    poetry config virtualenvs.create false && \
    poetry install

