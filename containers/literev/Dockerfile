# ref: https://github.com/mamba-org/micromamba-docker/blob/main/Dockerfile

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

ENV ENV_NAME=literev
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# we need to use a different approach
ENV UID=1000
ENV GID=1000

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

RUN addgroup --gid ${GID} literev \
  && useradd --uid ${UID} --gid ${GID} -ms /bin/bash literev \
  && mkdir -p /opt/services/literev \
  && chmod -R a+rwx /opt/conda /opt/services \
  && export ENV_NAME="$ENV_NAME" \
  && chown literev:literev /opt/services

USER literev

WORKDIR /opt/services/literev

COPY --chown=literev:literev ./conda/ /tmp/conda

RUN mamba env create -n $ENV_NAME --file /tmp/conda/base.yaml \
  && mamba env update -n $ENV_NAME --file /tmp/conda/service-linux-extra.yaml \
  && conda clean --all \
  && find /opt/conda/ -type f,l -name '*.pyc' -delete \
  && find /opt/conda/ -type f,l -name '*.js.map' -delete \
  && rm -rf /opt/conda/pkgs /tmp/*

ENV CONDA_PREFIX /opt/conda/envs/$ENV_NAME
ENV PATH ${CONDA_PREFIX}/bin:$PATH

# install dependencies
COPY --chown=literev:literev pyproject.toml /opt/services/literev

RUN poetry config virtualenvs.create false \
  && poetry config experimental.new-installer true \
  && poetry install --no-root --no-cache --only=main \
  && poetry cache clear pypi --all \
  && pip cache purge \
  && rm -rf ~/.cache/pypoetry/{cache,artifacts}

USER root

RUN touch /opt/entrypoint.sh \
  && chown literev.literev /opt/entrypoint.sh \
  && chmod +x /opt/entrypoint.sh \
  && echo "source /opt/entrypoint.sh" > ~/.bashrc

USER literev

ENV PYTHONPATH='/opt/services/literev'

ENTRYPOINT ["tini", "--", "/opt/entrypoint.sh"]
