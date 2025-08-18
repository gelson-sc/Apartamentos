FROM docker.io/python:3.13.5-slim-bookworm AS python

# Stage 1: Build our dependency wheels
FROM python AS python-build-stage

ARG BUILD_ENVIRONMENT=local

RUN apt-get update && apt-get install --no-install-recommends -y \
  build-essential \
  libpq-dev

COPY ./requirements /requeirements

RUN pip wheel --wheel-dir /usr/src/app/wheels \
  -r requeirements/${BUILD_ENVIRONMENT}.txt

# Stage 2: Python run stage.
FROM python AS python-run-stage
ARG BUILD_ENVIRONMENT=local
ARG APP_HOME=/app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV BUILD_ENV=${BUILD_ENVIRONMENT}


WORKDIR ${APP_HOME}
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# Combine package installations into a single RUN command to reduce image layers
RUN apt-get update && apt-get install --no-install-recommends -y \
  # For system management and debugging
  gnupg \
  sudo \
  git \
  bash-completion \
  nano \
  ssh \
  # For python dependencies (psycopg2)
  libpq-dev \
  # For Django translations
  gettext \
  && rm -rf /var/lib/apt/lists/*

RUN addgroup --system django && \
  adduser --system --ingroup django django && \
  mkdir -p ${APP_HOME}/staticfiles && \
  chown django:django ${APP_HOME}/staticfiles && \
  chmod 775 ${APP_HOME}/staticfiles -R  && \
  chmod 777 ./staticfiles -R

COPY --from=python-build-stage /usr/src/app/wheels /wheels/

RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
  && rm -rf /wheels/


# Copy all entry scripts at once
COPY --chown=django:django ./docker/local/django/entrypoint.sh /entrypoint
COPY --chown=django:django ./docker/local/django/start.sh /start
COPY --chown=django:django ./docker/local/django/celery/worker/start.sh /start-celeryworker
COPY --chown=django:django ./docker/local/django/celery/beat/start.sh /start-celerybeat
COPY --chown=django:django ./docker/local/django/celery/flower/start.sh /start-flower

# Fix line endings and make all scripts executable in a single RUN command
RUN sed -i 's/\r$//g' /entrypoint /start /start-celeryworker /start-celerybeat /start-flower && \
    chmod +x /entrypoint /start /start-celeryworker /start-celerybeat /start-flower

COPY --chown=django:django . ${APP_HOME}
RUN mkdir -p ${APP_HOME}/staticfiles/admin && \
    mkdir -p ${APP_HOME}/staticfiles/rest_framework && \
    mkdir -p ${APP_HOME}/staticfiles/drf-yasg


# RUN chown django:django ${APP_HOME}
RUN chown -R django:django ${APP_HOME}/staticfiles && chmod -R 777 ${APP_HOME}/staticfiles
USER django
#RUN pip install --no-cache-dir -r ./requirements/local.txt

ENTRYPOINT [ "/entrypoint" ]