FROM python:3.12-slim-bullseye
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
COPY --from=ghcr.io/astral-sh/uv:latest /uvx /bin/uvx

WORKDIR /opt/

# Install deps at /opt to avoid mountint host .venv

# Copy uv.lock* in case it doesn't exist in the repo
# COPY ./pyproject.toml README.md /uv.lock /opt/
# COPY ./app /opt/app

# Allow installing dev dependencies to run tests
# ARG INSTALL_DEV=false
# RUN bash -c "if [ $INSTALL_DEV == 'true' ] ; then uv sync ; else uv sync --only main ; fi"

RUN --mount=type=cache,target=/.cache/uv \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    --mount=type=bind,source=README.md,target=README.md \
    --mount=type=bind,source=app,target=app \
    uv sync
ENV VIRTUAL_ENV=/opt/.venv
ENV PATH="/opt/.venv/bin:$PATH"

WORKDIR /app/

COPY ./pyproject.toml /app/
COPY ./app /app/app
COPY README.md /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app

COPY ./start-reload.sh /start-reload.sh
RUN chmod +x /start-reload.sh

COPY ./scripts/ /app/

COPY ./alembic.ini /app/

COPY ./prestart.sh /app/
RUN chmod +x /app/prestart.sh

COPY ./tests-start.sh /app/
RUN chmod +x /app/tests-start.sh

EXPOSE 80

CMD ["/start-reload.sh"]
