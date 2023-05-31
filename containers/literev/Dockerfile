FROM condaforge/mambaforge:latest

LABEL maintainer="Ivan Ogasawara <ivan.ogasawara@gmail.com>"
LABEL org.opencontainers.image.title="LiteRev"
LABEL org.opencontainers.image.authors="LiteRev Team"
LABEL org.opencontainers.image.source="https://github.com/thegraphnetwork-literev/literev-docker-base"
LABEL org.opencontainers.image.version="latest"
LABEL org.opencontainers.image.description="LiteRev"
LABEL org.thegraphnetwork.literev.version="latest"

# it is the default, but using it here to have it explicitly
USER root

SHELL ["/bin/bash", "-c"]

ENV TZ=Etc/UTC
ENV ENV_NAME=literev
ENV DEBIAN_FRONTEND=noninteractive
ENV CONDA_PREFIX /opt/conda/envs/$ENV_NAME
ENV PATH ${CONDA_PREFIX}/bin:$PATH
ENV PYTHONPATH='/opt/services/literev'

# [Deps] apt-get
RUN apt-get update -y \
  && apt-get install -y \
    apt-utils \
    build-essential \
    curl \
    tini \
    tzdata \
    gcc-multilib \
    g++-multilib \
    openssl \
  && rm -rf /var/lib/apt/lists/* \
    /var/cache/apt/archives \
    /tmp/*

# [System] Configure non-root user
ARG UID
ARG GID
RUN set -ex \
    && if [ ! "${UID}" ] || [ ! "${GID}" ]; then \
      echo "UID and GID are not properly defined, pass "; \
      echo "them as arguments to build the image."; \
      exit 1; \
    else \
      addgroup --gid ${GID} literev \
      && useradd --uid ${UID} --gid ${GID} -ms /bin/bash literev; \
    fi

COPY --chown=literev:literev ./scripts/ /opt/scripts
COPY --chown=literev:literev ./conda/ /tmp/conda
COPY --chown=literev:literev pyproject.toml /opt/services/literev/

RUN chmod -R a+rwx /opt/conda /opt/services /opt/scripts \
  && export ENV_NAME="$ENV_NAME" \
  && echo "source /opt/scripts/entrypoint.sh" > ~/.bashrc

USER literev

WORKDIR /opt/services/literev

# [Deps] conda
RUN conda env create --file /tmp/conda/base.yaml \
  && conda env update --file /tmp/conda/service-linux-extra.yaml \
  && conda clean --all \
  && find /opt/conda/ -type f,l -name '*.pyc' -delete \
  && find /opt/conda/ -type f,l -name '*.js.map' -delete \
  && rm -rf /opt/conda/pkgs /tmp/*


# [Deps] poetry
RUN poetry config virtualenvs.create false \
  && poetry config experimental.new-installer true \
  && poetry install --no-root --no-cache --only=main \
  && poetry cache clear pypi --all \
  && pip cache purge \
  && rm -rf ~/.cache/pypoetry/{cache,artifacts}


ENTRYPOINT ["tini", "--", "/opt/scripts/entrypoint.sh"]
