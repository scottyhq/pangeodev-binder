FROM ubuntu:18.04

# RUN this section as root
# -----
ARG CONDA_VERSION=4.7.12.1
ARG NB_USER=jovyan
ARG NB_UID=1000

# Environment variables
ENV SHELL /bin/bash
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV CONDA_DIR /srv/conda
ENV NB_PYTHON_PREFIX ${CONDA_DIR}
ENV USER ${NB_USER}
ENV HOME /home/${USER}

# Copy auxiliary files
COPY . ${HOME}

WORKDIR ${HOME}

# Apt.txt Packages
RUN [ -d "binder" ] && cd binder \
    && apt-get update --fix-missing \
    && xargs -a apt.txt apt-get install -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create group + user, and set proper permissions on home & conda directories
RUN groupadd \
        --gid ${NB_UID} \
        ${NB_USER}  \
    && useradd \
        --comment "Default user" \
        --create-home \
        --gid ${NB_UID} \
        --no-log-init \
        --shell /bin/bash \
        --uid ${NB_UID} \
        ${NB_USER} 
        
# Move things around filesystem and change ownership
RUN [ -d "binder" ] && cd binder \
    && cp repo2docker-entrypoint /usr/local/bin/repo2docker-entrypoint \
    && mkdir ${CONDA_DIR} \
    && ln -s ${CONDA_DIR}/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
    && chown -R ${NB_USER}:${NB_USER} ${CONDA_DIR} \
    && chown -R ${NB_USER}:${NB_USER} ${HOME}


# RUN this section as NB_USER 
# -----
USER ${NB_USER}

# Install miniconda and configure conda
ENV PATH ${CONDA_DIR}/bin:$PATH
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-${CONDA_VERSION}-Linux-x86_64.sh -O ~/miniconda.sh \
    && /bin/bash ~/miniconda.sh -u -b -p ${CONDA_DIR} \
    && rm ~/miniconda.sh \
    && conda clean -afy \
    && conda config --add channels conda-forge \
    && conda config --set channel_priority strict \
    && conda config --system --set show_channel_urls true \
    && echo "conda activate notebook" >> ${CONDA_DIR}/etc/profile.d/conda.sh \
    && find ${CONDA_DIR} -follow -type f -name '*.a' -delete \
    && find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete

# Creat "notebook" environment from environment.yml and cleanup
RUN [ -d "binder" ] && cd binder \
    && conda env create -f environment.yml \
    && conda clean -afy \
    && find ${CONDA_DIR} -follow -type f -name '*.a' -delete \
    && find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete \
    && find ${CONDA_DIR} -follow -type f -name '*.js.map' -delete \
    && find ${CONDA_DIR}/envs/notebook/lib/python*/site-packages/bokeh/server/static -follow -type f -name '*.js' ! -name '*.min.js' -delete

# Run postBuild script
ENV PATH ${CONDA_DIR}/envs/notebook/bin:$PATH
RUN [ -d "binder" ] && cd binder \
    && ./postBuild \
    && mkdir -p ${NB_PYTHON_PREFIX}/etc/dask \
    && cp dask_config.yaml ${NB_PYTHON_PREFIX}/etc/dask/dask.yaml \
    && rm -rf ${HOME}/.cache ${HOME}/.npm ${HOME}/.yarn \
    && rm -rf ${NB_PYTHON_PREFIX}/share/jupyter/lab/staging \
    && find ${NB_PYTHON_PREFIX} -follow -type f -name '*.js.map' -delete

# Add start script

EXPOSE 8888

ENTRYPOINT ["/usr/local/bin/repo2docker-entrypoint"]
CMD ["jupyter", "notebook", "--ip", "0.0.0.0"]
