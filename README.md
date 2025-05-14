# Multi-Version Go Dev Container

Develop **multiple Go projects that require different compiler versions** without polluting your host machine.

---

## Why bother...

* **Error-free builds** – each project compiles against the exact Go tool-chain it needs.  
* **Fast switching** – change versions with a single command (`use_go_version <version>`).  
* **Clean host** – every tool-chain lives inside the container under `/workspace/<version>`; your local `/usr/local/go` is untouched.  
* **Disposable** – tear the container down when you’re done; all build artefacts disappear with it.

---

## How it works

1. Using VSCode devcontainers, the Dockerfile installs a helper called **`use_go_version`**.  
2. You unpack any Go release into `/workspace/<version>/`.  
3. Running `use_go_version <version>` repoints `/usr/local/go` to that folder and updates your `$PATH` instantly.  
4. Your current project is bind-mounted into `/workspace`, so Git operations (push/pull) work as usual with your host credentials.

```bash
# Try it yourself! Download + untar Go 1.23.2 straight into that folder
wget -O - https://go.dev/dl/go1.23.2.linux-amd64.tar.gz | tar -xz --strip-components=1 -C /workspace/go1.23.2
use_go_version <your_new_version>
go version
```
---

## Configurations 

### `.devcontainer/devcontainer.json`

```json
{
  "name": "Golang Multi-Version Environment",
  "dockerFile": "Dockerfile",
  "context": "..",
  "platform": "linux/amd64",
  "workspaceFolder": "/workspace",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind",
  "settings": {
    "terminal.integrated.shell.linux": "/bin/bash"
  },
  "extensions": [
    "golang.Go"
  ]
}
```

### `.devcontainer/Dockerfile`

```Dockerfile
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
```