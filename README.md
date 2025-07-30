# Go and Node.js Version Manager for Ubuntu

This repository provides two Zsh functions to manage Go and Node.js installations on Ubuntu 24.04. The functions allow you to install or update to the latest versions of Go and Node.js (LTS) effortlessly.

## Functions

- **`setup_go`**: Installs or updates Go to the latest stable version.
- **`setup_node`**: Installs or updates Node.js to the latest Long-Term Support (LTS) version using Node Version Manager (nvm).
- **`setup_python`**: Installs or updates Python to the latest stable version using Python Version Manager (pyenv).

## Prerequisites

Before using the functions, ensure the following tools are installed on your Ubuntu 24.04 system:

```bash
sudo apt update
sudo apt install curl wget tar -y
```

- **`curl`**: Used to fetch version information and download nvm.
- **`wget`**: Used to download the Go tarball.
- **`tar`**: Used to extract the Go tarball.

Additionally, you need:
- A working Zsh shell (default in Ubuntu 24.04).
- `sudo` privileges for installing Go to `/usr/local/go`.

## Installation

1. **Add the functions to your `~/.zshrc`:**
   - Open your Zsh configuration file in a text editor (e.g., `nano`):
     ```bash
     nano ~/.zshrc
     ```
   - Copy and paste the following code at the end of the file:

     ```zsh
     # Function to install or update Go to the latest version
     setup_go() {
       # Ensure curl is installed
       if ! command -v curl >/dev/null 2>&1; then
         echo "Error: curl is required. Install it with 'sudo apt install curl'"
         return 1
       fi

       # Get the latest Go version from the official site
       LATEST_GO=$(curl -sSL "https://go.dev/VERSION?m=text" | head -n 1 | grep -E '^go[0-9]+\.[0-9]+\.[0-9]+$')
       if [ -z "$LATEST_GO" ]; then
         echo "Error: Failed to fetch latest Go version. Check your internet connection or try again later."
         echo "Debug: Run 'curl -sSL \"https://go.dev/VERSION?m=text\"' to diagnose."
         return 1
       fi

       CURRENT_GO=""
       # Check if Go is installed
       if command -v go >/dev/null 2>&1; then
         CURRENT_GO=$(go version | awk '{print $3}')
       fi

       if [ "$LATEST_GO" = "$CURRENT_GO" ]; then
         echo "Go is already up-to-date: $CURRENT_GO"
         return
       fi

       echo "Installing/Updating Go to $LATEST_GO..."

       # Download and install Go
       if ! wget "https://go.dev/dl/$LATEST_GO.linux-amd64.tar.gz" -O /tmp/go.tar.gz; then
         echo "Error: Failed to download Go $LATEST_GO. Check your internet connection."
         return 1
       fi
       sudo rm -rf /usr/local/go
       if ! sudo tar -C /usr/local -xzf /tmp/go.tar.gz; then
         echo "Error: Failed to extract Go tarball."
         rm /tmp/go.tar.gz
         return 1
       fi
       rm /tmp/go.tar.gz

       # Ensure Go binary is in PATH
       if ! grep -q "/usr/local/go/bin" ~/.zshrc; then
         echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.zshrc
       fi

       # Update PATH for current session
       export PATH=$PATH:/usr/local/go/bin

       # Verify installation
       if /usr/local/go/bin/go version >/dev/null 2>&1; then
         echo "Go $LATEST_GO installed successfully"
       else
         echo "Go installation failed"
         return 1
       fi
     }

     # Function to install or update Node.js to the latest LTS version using nvm
     setup_node() {
       # Ensure curl is installed
       if ! command -v curl >/dev/null 2>&1; then
         echo "Error: curl is required. Install it with 'sudo apt install curl'"
         return 1
       fi

       # Install nvm if not already installed
       if [ ! -d "$HOME/.nvm" ]; then
         echo "Installing nvm..."
         if ! curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash; then
           echo "Error: Failed to install nvm. Check your internet connection."
           return 1
         fi
       fi

       # Load nvm
       export NVM_DIR="$HOME/.nvm"
       [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
       if [ $? -ne 0 ]; then
         echo "Error: Failed to load nvm. Check ~/.nvm/nvm.sh existence and permissions."
         return 1
       fi

       # Get the latest Node.js LTS version
       LATEST_NODE=$(nvm ls-remote --lts | grep -E 'v[0-9]+\.[0-9]+\.[0-9]+' | tail -n 1 | awk '{print $1}' | tr -d '[:space:]')
       if [ -z "$LATEST_NODE" ]; then
         echo "Error: Failed to fetch latest Node.js LTS version."
         echo "Debug: Run 'nvm ls-remote --lts' to check available versions."
         nvm ls-remote --lts
         return 1
       fi

       CURRENT_NODE=""
       # Check if Node.js is installed
       if command -v node >/dev/null 2>&1; then
         CURRENT_NODE=$(node -v)
       fi

       if [ "$LATEST_NODE" = "$CURRENT_NODE" ]; then
         echo "Node.js is already up-to-date: $CURRENT_NODE"
         return
       fi

       echo "Installing/Updating Node.js to $LATEST_NODE..."

       # Install the latest Node.js LTS version
       if ! nvm install "$LATEST_NODE"; then
         echo "Error: Failed to install Node.js $LATEST_NODE."
         return 1
       fi
       nvm use "$LATEST_NODE"
       nvm alias default "$LATEST_NODE"

       # Verify installation
       if node -v >/dev/null 2>&1; then
         echo "Node.js $LATEST_NODE installed successfully"
       else
         echo "Node.js installation failed"
         return 1
       fi
     }

     # Function to install or update Python to the latest version using pyenv
      setup_python() {
        # Prerequisites for building Python from source
        echo "Installing Python build dependencies..."
        sudo apt update
        sudo apt install -y \
          build-essential \
          zlib1g-dev \
          libncurses5-dev \
          libgdbm-dev \
          libnss3-dev \
          libssl-dev \
          libreadline-dev \
          libffi-dev \
          libsqlite3-dev \
          libbz2-dev \
          liblzma-dev \
          tk-dev \
          libexpat1-dev
      
        # Install pyenv if not already installed
        if [ ! -d "$HOME/.pyenv" ]; then
          echo "Installing pyenv..."
          if ! curl https://pyenv.run | bash; then
            echo "Error: Failed to install pyenv. Check your internet connection."
            return 1
          fi
          echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
          echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
          echo 'eval "$(pyenv init -)"' >> ~/.zshrc
        fi
      
        # Load pyenv for the current session
        export PYENV_ROOT="$HOME/.pyenv"
        export PATH="$PYENV_ROOT/bin:$PATH"
        eval "$(pyenv init -)"
      
        # Check if pyenv is loaded
        if ! command -v pyenv >/dev/null 2>&1; then
          echo "Error: pyenv is not loaded. Try restarting your shell or running 'source ~/.zshrc'."
          return 1
        fi
      
        # Get the latest stable Python version
        LATEST_PYTHON=$(pyenv install --list | grep -E '^\s+[0-9]+\.[0-9]+\.[0-9]+$' | tail -n 1 | tr -d '[:space:]')
        if [ -z "$LATEST_PYTHON" ]; then
          echo "Error: Failed to fetch latest Python version."
          echo "Debug: Run 'pyenv install --list' to check available versions."
          return 1
        fi
      
        # Check if the latest version is already installed
        if pyenv versions | grep -q "$LATEST_PYTHON"; then
          echo "Python $LATEST_PYTHON is already installed."
        else
          echo "Installing Python $LATEST_PYTHON..."
          if ! pyenv install "$LATEST_PYTHON"; then
            echo "Error: Failed to install Python $LATEST_PYTHON."
            echo "You may need to install additional build dependencies. See pyenv documentation."
            return 1
          fi
        fi
      
        # Set the installed version as the global default
        if ! pyenv global "$LATEST_PYTHON"; then
          echo "Error: Failed to set Python $LATEST_PYTHON as the global default."
          return 1
        fi
        
        echo "Python $LATEST_PYTHON installed and set as the global default successfully."
        echo "To install other versions, use 'pyenv install <version>'."
      }
     ```

   - Save the file (`Ctrl+O`, `Enter`, then `Ctrl+X` in nano).

2. **Reload your Zsh configuration:**
   ```bash
   source ~/.zshrc
   ```

## Usage

Run the functions directly in your terminal:

- **To install or update Go:**
  ```bash
  setup_go
  ```
  This checks for the latest Go version, installs it if not present, or updates it if outdated.

- **To install or update Node.js:**
  ```bash
  setup_node
  ```
  This installs `nvm` (if not already installed) and uses it to install or update to the latest Node.js LTS version.

## Notes

- **Go Installation:**
  - The `setup_go` function assumes an `amd64` architecture. For other architectures (e.g., `arm64`), modify the `wget` URL in the function to match the correct tarball from `https://go.dev/dl/`.
  - Requires `sudo` privileges to install Go to `/usr/local/go`.
  - The Go binary path (`/usr/local/go/bin`) is added to your `PATH` automatically.

- **Node.js Installation:**
  - The `setup_node` function uses `nvm` to manage Node.js versions, ensuring compatibility and easy version switching.
  - It installs the latest LTS version of Node.js for stability.
  - The default Node.js version is set to the latest LTS after installation.

- **Python Installation:**
   - The setup_python function uses pyenv to manage Python versions, which is highly recommended for keeping your system's Python installation clean.
   - It installs the latest stable Python version available and sets it as the global default.
   - Python is installed from source, which requires some build dependencies. These are automatically installed by the function.
   - Once pyenv is installed, you can use its commands to manage versions, create virtual environments, and switch between them.

- **Verification:**
  - All three functions verify the installation by checking the installed version.
  - If an installation fails, an error message is displayed.

- **Manual Version Check:**
  - For Go: Visit `https://go.dev/dl/` or run `curl -sSL "https://go.dev/VERSION?m=text"` to see the latest version.
  - For Node.js: Run `nvm ls-remote --lts` to list available Node.js LTS versions.
  - For Python: Run `pyenv install --list` to see all available Python versions.

## Troubleshooting

- **Permission Issues:** Ensure you have `sudo` access for Go installation.
- **Network Issues:** Verify internet connectivity for downloading Go or nvm.
- **Version Manager Not Found:** If `nvm` or `pyenv` commands fail, ensure the `source ~/.zshrc` command was run after adding the functions.
- **Architecture Mismatch:** If using a non-`amd64` system, update the Go download URL in `setup_go`.
- **Version Fetch Errors:** If version fetching fails, manually check versions with `curl -sSL "https://go.dev/VERSION?m=text"` for Go or `nvm ls-remote --lts` for Node.js. If `nvm ls-remote --lts` is empty, try reinstalling nvm with:
  ```bash
  rm -rf ~/.nvm
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
  source ~/.zshrc
  ```

## Contributing

Feel free to submit issues or pull requests to improve these functions or add support for other tools!
