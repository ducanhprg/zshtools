# Go and Node.js Version Manager for Ubuntu

This repository provides two Zsh functions to manage Go and Node.js installations on Ubuntu 24.04. The functions allow you to install or update to the latest versions of Go and Node.js (LTS) effortlessly.

## Functions

- **`setup_go`**: Installs or updates Go to the latest stable version.
- **`setup_node`**: Installs or updates Node.js to the latest Long-Term Support (LTS) version using Node Version Manager (nvm).

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
       # Get the latest Go version from the official site
       LATEST_GO=$(curl -s https://go.dev/VERSION?m=text | head -n 1)
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
       wget https://go.dev/dl/$LATEST_GO.linux-amd64.tar.gz -O /tmp/go.tar.gz
       sudo rm -rf /usr/local/go
       sudo tar -C /usr/local -xzf /tmp/go.tar.gz
       rm /tmp/go.tar.gz

       # Ensure Go binary is in PATH
       if ! grep -q "/usr/local/go/bin" ~/.zshrc; then
         echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.zshrc
       fi

       # Verify installation
       if /usr/local/go/bin/go version >/dev/null 2>&1; then
         echo "Go $LATEST_GO installed successfully"
         source ~/.zshrc
       else
         echo "Go installation failed"
         return 1
       fi
     }

     # Function to install or update Node.js to the latest LTS version using nvm
     setup_node() {
       # Install nvm if not already installed
       if [ ! -d "$HOME/.nvm" ]; then
         echo "Installing nvm..."
         curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
         
         # Load nvm
         export NVM_DIR="$HOME/.nvm"
         [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
       else
         # Load nvm
         export NVM_DIR="$HOME/.nvm"
         [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
       fi

       # Get the latest Node.js LTS version
       LATEST_NODE=$(nvm ls-remote | grep "Latest LTS" | tail -n 1 | awk '{print $2}')
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
       nvm install $LATEST_NODE
       nvm use $LATEST_NODE
       nvm alias default $LATEST_NODE

       # Verify installation
       if node -v >/dev/null 2>&1; then
         echo "Node.js $LATEST_NODE installed successfully"
       else
         echo "Node.js installation failed"
         return 1
       fi
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

- **Verification:**
  - Both functions verify the installation by checking the installed version.
  - If an installation fails, an error message is displayed.

- **Manual Version Check:**
  - For Go: Visit `https://go.dev/dl/` to see available versions.
  - For Node.js: Run `nvm ls-remote` to list available Node.js versions.

## Troubleshooting

- **Permission Issues:** Ensure you have `sudo` access for Go installation.
- **Network Issues:** Verify internet connectivity for downloading Go or nvm.
- **nvm Not Found:** If `nvm` commands fail, ensure the `source ~/.zshrc` command was run after adding the functions.
- **Architecture Mismatch:** If using a non-`amd64` system, update the Go download URL in `setup_go`.

## Contributing

Feel free to submit issues or pull requests to improve these functions or add support for other tools!
