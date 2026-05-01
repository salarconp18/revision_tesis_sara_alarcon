ARG BASE_IMAGE=quay.io/jupyter/scipy-notebook:2025-12-31
FROM ${BASE_IMAGE}

ARG env_name=sara_semseg

COPY --chown=${NB_UID}:${NB_GID} environment.yml /tmp/environment.yml

# Crear el entorno con conda en lugar de mamba
RUN conda env create -p "${CONDA_DIR}/envs/${env_name}" -f /tmp/environment.yml && \
    conda clean --all -f -y && \
    "${CONDA_DIR}/envs/${env_name}/bin/python" -m ipykernel install \
      --user \
      --name="${env_name}" \
      --display-name="Python (${env_name})" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# Instalar PyTorch dentro del entorno
RUN "${CONDA_DIR}/envs/${env_name}/bin/pip" install --no-cache-dir \
    torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128

# Activar el entorno por defecto en Jupyter
RUN /opt/setup-scripts/activate_notebook_custom_env.py "${env_name}"

WORKDIR /home/jovyan/work