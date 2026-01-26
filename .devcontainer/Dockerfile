# 1. Base Image Selection
# We start with a pre-built Ubuntu 22.04 image optimized for dev containers.
# This provides a standard Linux environment with common developer tools (git, zsh, etc.) pre-installed.
FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

# 2. Install System Dependencies
# 'wget' is required to download the Anaconda installer script from the web.
# We update the package list (apt-get update), install wget, and then clean up
# the apt cache to keep the image size small.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends wget \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Environment Variables
# We define where Anaconda will be installed. keeping it in /usr/local ensures it's globally accessible.
ENV ANACONDA_PATH=/usr/local/anaconda3
# We add the Anaconda 'bin' directory to the system PATH so we can type 'conda' or 'jupyter' anywhere.
ENV PATH=$ANACONDA_PATH/bin:$PATH

# 4. Dynamic Anaconda Installation
# This block detects the machine's architecture to download the correct installer.
# - 'uname -m' tells us if we are on Intel (x86_64) or Apple Silicon (aarch64).
# - We set the ANACONDA_INSTALLER filename accordingly.
# - We then download the script to /tmp, run it in batch mode (-b) to the specified path (-p),
#   and delete the installer to save space.
RUN ARCH=$(uname -m) \
    && if [ "$ARCH" = "x86_64" ]; then \
        ANACONDA_INSTALLER="Anaconda3-2025.12-2-Linux-x86_64.sh"; \
    elif [ "$ARCH" = "aarch64" ]; then \
        ANACONDA_INSTALLER="Anaconda3-2025.12-2-Linux-aarch64.sh"; \
    else \
        echo "Unsupported architecture: $ARCH"; exit 1; \
    fi \
    && wget https://repo.anaconda.com/archive/$ANACONDA_INSTALLER -O /tmp/anaconda.sh \
    && bash /tmp/anaconda.sh -b -p $ANACONDA_PATH \
    && rm /tmp/anaconda.sh

# 5. Accept Terms of Service
# Newer versions of Anaconda require explicit acceptance of their Terms of Service.
# This command prevents the "CondaToSNonInteractiveError" during package installation.
RUN conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main \
    && conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

# 6. Install Jupyter
# We install the 'jupyter' package using conda.
# The '-y' flag automatically says "yes" to prompts.
RUN conda install -y jupyter

# 7. Fix Permissions
# The container runs as the 'vscode' user by default for development.
# We must ensure this user owns the Anaconda directory so they can install new packages
# or create environments without "Permission denied" errors.
RUN chown -R vscode:vscode $ANACONDA_PATH \
    && chmod -R 775 $ANACONDA_PATH

# 8. Expose Ports
# We document that the container listens on port 8888 (standard for Jupyter).
EXPOSE 8888
