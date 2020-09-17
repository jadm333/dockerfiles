# Source: https://github.com/jupyter/docker-stacks/blob/master/datascience-notebook/Dockerfile
ARG BASE_CONTAINER=jupyter/datascience-notebook
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

WORKDIR $HOME

# docker build -t jadm333/ds_lab .
# Flish shell
# docker run --name ds_lab -it -p 8888:8888 -e JUPYTER_ENABLE_LAB=yes -e NB_UID=(id -u) -e NB_GID=(id -g) -v "$HOME/Documents":/home/jovyan/work jadm333/ds_lab