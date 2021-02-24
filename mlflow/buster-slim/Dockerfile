ARG PYTHON_VERSION

FROM python:${PYTHON_VERSION}-slim-buster

ARG TINI_VERSION
ARG CONDA_VERSION
ARG MLFLOW_VERSION

ARG MLFLOW_USER=mlflow
ARG MLFLOW_UID=1000

RUN useradd --uid "${MLFLOW_UID}" \
            --home-dir /mlflow \
            --create-home "${MLFLOW_USER}"

RUN apt-get update \
 && apt-get install --upgrade --yes --no-install-recommends \
    ca-certificates \
    curl \
    git \
    tzdata \
 && apt-get autoremove --yes --purge \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && update-ca-certificates \
 && ln -sf /usr/share/zoneinfo/UTC /etc/localtime

# Miniconda latest
RUN curl -L -o /tmp/miniconda.sh "https://repo.anaconda.com/miniconda/Miniconda3-${CONDA_VERSION}-Linux-x86_64.sh" \
 && sh /tmp/miniconda.sh -b -p /opt/conda \
 && rm -rf /tmp/miniconda.sh \
 && export PATH="/opt/conda/bin:${PATH}" \
 && ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
 && echo ". /opt/conda/etc/profile.d/conda.sh" >> /mlflow/.bashrc \
 && find /opt/conda/ -follow -type f -name '*.a' -delete \
 && find /opt/conda/ -follow -type f -name '*.js.map' -delete \
 && /opt/conda/bin/conda clean -afy

# tini init
RUN curl -L -o /usr/bin/tini "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini-static-amd64" \
 && chown root:root /usr/bin/tini \
 && chmod 0755 /usr/bin/tini \
 && /usr/bin/tini --version

# MLflow
RUN pip install --upgrade \
    mlflow[extras]=="${MLFLOW_VERSION}" \
 && mlflow --version

USER "${MLFLOW_USER}"
WORKDIR /mlflow

ENTRYPOINT ["/usr/bin/tini", "--", "mlflow"]
CMD ["server"]
