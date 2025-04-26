FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Set working directory
ENV WORKDIR=/workspace
WORKDIR $WORKDIR

# Install system dependencies for R and Python packages
# Including libraries needed for Seurat, Bioconductor, Scanpy, etc.
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    # Basic utilities
    sudo \
    build-essential \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    curl \
    wget \
    gnupg \
    # R dependencies
    libssl-dev \
    libcurl4-openssl-dev \
    libxml2-dev \
    libfontconfig1-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    libhdf5-dev \
    # Python dependencies (some might be needed by R packages too)
    libopenblas-dev \
    liblapack-dev && \
    # Clean up apt cache
    rm -rf /var/lib/apt/lists/*

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/

# Switch to vscode user
USER vscode

# --- Install R ---
# Add CRAN repository and install R
RUN wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
RUN sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/'
RUN --mount=type=cache,target=/var/cache/apt \
    sudo apt-get update && \
    sudo apt-get install -y --no-install-recommends r-base r-base-dev && \
    sudo rm -rf /var/lib/apt/lists/*

ENV R_LIBS_USER=/home/vscode/R/library
ENV R_DEFAULT_REPOS=https://cloud.r-project.org/
RUN mkdir -p "$R_LIBS_USER"
RUN printf 'R_LIBS_USER=%s\n' "$R_LIBS_USER" >> ~/.Renviron
RUN printf '.libPaths(Sys.getenv("R_LIBS_USER"))\n' >> ~/.Rprofile

# Install common R packages for single-cell analysis
# Using littler for non-interactive installation
RUN --mount=type=cache,target=/home/vscode/.cache/R \
    R -e "install.packages('remotes', repos='https://cloud.r-project.org/')" && \
    R -e "install.packages(c('tidyverse', 'devtools', 'BiocManager', 'Matrix', 'irlba', 'igraph', 'patchwork', 'ggplot2', 'dplyr', 'hdf5r'), repos=Sys.getenv('R_DEFAULT_REPOS'), Ncpus=parallel::detectCores())" && \
    # Install Bioconductor core packages
    R -e "BiocManager::install(c('BiocGenerics', 'GenomeInfoDb', 'GenomicRanges', 'SummarizedExperiment', 'SingleCellExperiment', 'zellkonverter'), update=FALSE, ask=FALSE, Ncpus=parallel::detectCores())" && \
    # Install Seurat and SeuratDisk
    R -e "remotes::install_github('satijalab/seurat', 'seurat5', upgrade = 'never', Ncpus=parallel::detectCores())" && \
    R -e "remotes::install_github('mojaveazure/seurat-disk', upgrade = 'never', Ncpus=parallel::detectCores())" && \
    # Other utilities (languageserver, httpgd, IRkernel)
    R -e "remotes::install_github('REditorSupport/languageserver', Ncpus=parallel::detectCores())" && \
    R -e "remotes::install_github('nx10/httpgd', Ncpus=parallel::detectCores())"

RUN sudo chown -R vscode:vscode /workspace

# --- Install Python ---
# Install Python packages as vscode user
ENV PATH="$WORKDIR/.venv/bin:$PATH"
RUN uv venv --seed --python 3.12
RUN --mount=type=cache,target=/home/vscode/.cache/uv \
    uv pip install --no-cache-dir \
    numpy \
    scipy \
    pandas \
    anndata \
    scanpy \
    matplotlib \
    seaborn \
    scikit-learn \
    jupyterlab \
    ipykernel \
    radian

# Set back to interactive
ENV DEBIAN_FRONTEND=dialog

CMD ["/bin/bash"]
