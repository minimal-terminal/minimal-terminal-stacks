# Curated Tools: Advanced CLI Productivity Stack

Unlock unparalleled efficiency and control in your terminal with this curated stack of advanced command-line interface (CLI) tools. Designed for sysadmins, developers, and homelab enthusiasts, these utilities streamline complex tasks, enhance data visibility, and transform your command line into a true productivity powerhouse.

--- 

### 01. Lazydocker

*   **Description:** A powerful terminal UI for Docker, offering an intuitive way to manage containers, images, volumes, and networks with keyboard shortcuts.
*   **Project Link:** `https://github.com/jesseduffield/lazydocker`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install jesseduffield/lazydocker/lazydocker

    # Go (requires Go installed)
    go install github.com/jesseduffield/lazydocker@latest
    ```
*   **Docker Note:** Lazydocker is a host-based utility that interacts with your Docker daemon to provide a TUI for management. It is not typically run within a Docker container itself.

--- 

### 02. k9s

*   **Description:** A terminal UI to navigate, observe, and manage your Kubernetes clusters with ease, providing real-time insights into your cluster's state.
*   **Project Link:** `https://github.com/derailed/k9s`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install k9s

    # Linux (Snap)
    snap install k9s
    ```
*   **Docker Note:** k9s runs on your host machine, leveraging your `kubectl` configuration to manage remote Kubernetes clusters. It's a client-side tool.

--- 

### 03. bat

*   **Description:** A `cat` clone with syntax highlighting, Git integration, and automatic paging, making file viewing a pleasure with enhanced readability.
*   **Project Link:** `https://github.com/sharkdp/bat`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install bat

    # Debian/Ubuntu
    sudo apt install bat
    ```
*   **Docker Example:** View a file with syntax highlighting inside a temporary container.
    ```bash
    docker run --rm -v $(pwd):/data alpine/git:latest /bin/sh -c \
      "apk add bat && echo 'print(\"Hello, Docker!\")' > /data/hello.py && bat /data/hello.py"
    ```

--- 

### 04. fd

*   **Description:** A blazing fast and user-friendly alternative to `find`, with sensible defaults and parallel execution for quick and intuitive file searches.
*   **Project Link:** `https://github.com/sharkdp/fd`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install fd

    # Debian/Ubuntu
    sudo apt install fd-find
    ```
*   **Docker Example:** Find `.txt` files within a mounted volume inside a container.
    ```bash
    docker run --rm -v $(pwd):/data alpine/git:latest /bin/sh -c \
      "apk add fd && touch /data/file1.txt /data/file2.md && fd '.txt' /data"
    ```

--- 

### 05. ripgrep

*   **Description:** A super-fast line-oriented search tool that recursively searches directories for regex patterns, built for speed and efficiency over `grep`.
*   **Project Link:** `https://github.com/BurntSushi/ripgrep`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install ripgrep

    # Debian/Ubuntu
    sudo apt install ripgrep
    ```
*   **Docker Example:** Search for a pattern in a log file within a containerized environment.
    ```bash
    docker run --rm -v $(pwd):/data alpine/git:latest /bin/sh -c \
      "apk add ripgrep && echo 'search this line' > /data/log.txt && rg 'line' /data"
    ```

--- 

### 06. procs

*   **Description:** A modern replacement for `ps` that delivers enhanced process information in a colorful, human-readable format, improving system monitoring.
*   **Project Link:** `https://github.com/dalance/procs`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install procs

    # Rust (requires Cargo)
    cargo install procs
    ```
*   **Docker Example:** List host processes from within a privileged container.
    ```bash
    docker run --rm --pid=host --cap-add SYS_PTRACE alpine/git:latest /bin/sh -c \
      "apk add procs && procs"
    ```
    *Note: `--pid=host` and `--cap-add SYS_PTRACE` are required to see host processes.*

--- 

### 07. dog

*   **Description:** A user-friendly command-line DNS client, offering a colorful and intuitive alternative to `dig` for performing DNS lookups.
*   **Project Link:** `https://github.com/ogham/dog`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install dog

    # Debian/Ubuntu
    sudo apt install dog
    ```
*   **Docker Example:** Perform a DNS lookup for a domain from inside a container.
    ```bash
    docker run --rm alpine/git:latest /bin/sh -c \
      "apk add dog && dog google.com"
    ```

--- 

### 08. zellij

*   **Description:** A dynamic terminal workspace and multiplexer that lets you manage multiple panes, tabs, and sessions with ease, designed for modern workflows.
*   **Project Link:** `https://github.com/zellij-org/zellij`
*   **Installation:**
    ```bash
    # macOS (Homebrew)
    brew install zellij

    # Rust (requires Cargo)
    cargo install zellij
    ```
*   **Docker Example:** Launch zellij inside an interactive container session.
    ```bash
    docker run -it --rm alpine/git:latest /bin/sh -c \
      "apk add zellij && zellij"
    ```
    *Note: Requires an interactive terminal (`-it`) to use zellij effectively.*