FROM python:3.9-bookworm

# Set the working directory
WORKDIR /workspace

# Install git and other dev tools
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
        man-db \
        build-essential \
        gpg \
        curl \
        git \
        sqlite3 \
        openssh-client \
        pinentry-tty \
        pinentry-curses \
        jq \
        inetutils-ping \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install fish shell
RUN echo 'deb http://download.opensuse.org/repositories/shells:/fish:/release:/3/Debian_12/ /' | tee /etc/apt/sources.list.d/shells:fish:release:3.list && \
    curl -fsSL https://download.opensuse.org/repositories/shells:fish:release:3/Debian_12/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/shells_fish_release_3.gpg > /dev/null && \
    apt-get update && \
    apt-get install --no-install-recommends -y \
        fish \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install starship
RUN curl -sS https://starship.rs/install.sh | sh -s -- --force

# Config nonroot user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create nonroot user
RUN groupadd --gid $USER_GID $USERNAME && \
    useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /usr/bin/fish && \
    chown -R $USERNAME /workspace
USER $USERNAME

# Create folder for vscode extensions volume mount
RUN mkdir -p /home/$USERNAME/.vscode-server/extensions

# Setup shell
RUN mkdir -p ~/.config/fish/functions/ && \
    echo 'starship init fish | source' >> ~/.config/fish/config.fish && \
    echo 'command_timeout = 5000' >> ~/.config/starship.toml && \
    printf '[container]\ndisabled = true' >> ~/.config/starship.toml
COPY --chown=$USER_UID:$USER_GID .devcontainer/configs/*.fish /home/$USERNAME/.config/fish/functions/
RUN fish -c "fish_update_completions"

# Add rust
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y

# Set python-related envs
ENV PYTHONUNBUFFERED=1 \
    # prevents python creating .pyc files
    PYTHONDONTWRITEBYTECODE=1

# Install uv https://docs.astral.sh/uv/getting-started/installation/#standalone-installer
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
# Configure uv
RUN echo 'set -x UV_LINK_MODE copy' >> ~/.config/fish/config.fish
RUN echo 'set -x UV_PYTHON_PREFERENCE only-system' >> ~/.config/fish/config.fish
RUN echo 'set -x UV_PYTHON_DOWNLOADS never' >> ~/.config/fish/config.fish
RUN echo 'uv generate-shell-completion fish | source' >> ~/.config/fish/config.fish
