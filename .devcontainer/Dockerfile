# Base image for Go development
FROM debian:bullseye-slim

# Install common tools
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    tar \
    git \
    bash \
    libc6 \
    && apt-get clean

# Add a script to switch Go versions
RUN echo '#!/bin/bash\n'\
'use_go_version() {\n'\
'  if [ -z "$1" ]; then\n'\
'    echo "Usage: use_go_version <version>"; return;\n'\
'  fi\n'\
'  if [ -d "/workspace/$1" ]; then\n'\
'    ln -sfn /workspace/$1 /usr/local/go;\n'\
'    export PATH="/usr/local/go/bin:$PATH";\n'\
'    echo "Switched to Go version $1";\n'\
'  else\n'\
'    echo "Go version $1 not found in /workspace";\n'\
'  fi\n'\
'}\n' >> /etc/bash.bashrc

# Set default Go version to 1.22.0 (can be switched using the script)
ENV PATH="/usr/local/go/bin:$PATH"

# Set up workspace
WORKDIR /workspace

# Default command
CMD [ "bash" ]