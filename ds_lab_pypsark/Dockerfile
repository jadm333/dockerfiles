# Source: https://github.com/jupyter/docker-stacks/blob/master/datascience-notebook/Dockerfile
ARG BASE_CONTAINER=jupyter/pyspark-notebook
FROM $BASE_CONTAINER

RUN pip install nbdime
RUN pip install jupyterlab-git
RUN conda install --quiet --yes "notebook>=5.3" "ipywidgets>=7.2" && \
    conda clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
RUN jupyter labextension install @deathbeds/jupyterlab-fonts --no-build
RUN jupyter labextension install @deathbeds/jupyterlab-font-fira-code --no-build
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager --no-build
RUN jupyter labextension install @jupyterlab/toc --no-build
RUN jupyter labextension install @telamonian/theme-darcula --no-build
RUN jupyter labextension install jupyterlab-plotly --no-build
RUN jupyter labextension install jupyterlab-chart-editor
RUN jupyter labextension install plotlywidget@4.10.0

RUN jupyter lab build

USER $NB_UID

WORKDIR $HOME

# docker build -t jadm333/ds_lab_pyspark .