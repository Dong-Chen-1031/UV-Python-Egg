# Python with UV - Yolks Images & Pterodactyl/Pelican Eggs

A complete solution for running Python applications on Pterodactyl/Pelican panels with modern tooling and automated workflows, consisting of:

1. **Runtime Docker Images (Yolks)**: Python images (3.8-3.14) with UV package manager pre-installed for blazing-fast dependency management
2. **Installer Docker Image**: Dedicated build environment with root access and all compilation dependencies pre-installed
3. **Pterodactyl/Pelican Eggs**: Ready-to-use egg configurations based on [pelican-eggs/generic](https://github.com/pelican-eggs/generic), supporting both UV and traditional pip workflows
4. **Automated Egg Generation**: Intelligent hen.py generator that automatically maintains egg configurations from templates

This project provides a modern, high-performance Python development environment with automatic dependency management, zero-permission issues, and fully automated CI/CD workflows.

## Quick Start

### Import the Egg Configuration

Choose the egg file based on your panel type:

#### For Pelican Panel (Recommended)
```
https://raw.githubusercontent.com/Dong-Chen-1031/yolks-uv/refs/heads/master/egg/egg-python-uv.json
```

#### For Pterodactyl Panel
```
https://raw.githubusercontent.com/Dong-Chen-1031/yolks-uv/refs/heads/master/egg/egg-pterodactyl-python-uv.json
```

**Import Steps:**
1. In your admin panel, navigate to **Nests** → Select your nest → **Import Egg**
2. Paste the appropriate URL above in the import field
3. Click import and the egg will be automatically configured

**Alternative Downloads:**
- [Pelican: egg-python-uv.json](https://raw.githubusercontent.com/Dong-Chen-1031/yolks-uv/refs/heads/master/egg/egg-python-uv.json)
- [Pterodactyl: egg-pterodactyl-python-uv.json](https://raw.githubusercontent.com/Dong-Chen-1031/yolks-uv/refs/heads/master/egg/egg-pterodactyl-python-uv.json)

### Deploy Your Application

After importing the egg:
1. Create a new server using the "Python with UV" egg
2. Select your desired Python version from the Docker images dropdown
3. Configure your application:
   - **Git Repo Address**: Your repository URL
   - **App py file**: Your main Python file (default: `app.py`)
   - **Requirements file**: Your requirements file (default: `requirements.txt`)
4. Start your server and the application will automatically deploy

## Components

### 1. Runtime Docker Images (Yolks)
Pre-built Python images (3.8-3.14) with UV package manager, hosted on GitHub Container Registry. These lightweight images run as non-root user (`container:container`) for enhanced security and serve as the runtime environment for your Python applications.

**Image URLs**: `ghcr.io/dong-chen-1031/yolks:python_uv_3.{8-14}`

### 2. Installer Docker Image
Dedicated installation-time container that runs with root privileges and includes all build dependencies pre-installed. This eliminates permission issues during package compilation and removes the need for apt commands in installation scripts.

**Image URL**: `ghcr.io/dong-chen-1031/yolks:python_uv_installer`

**Pre-installed Dependencies**:
- Base tools: git, curl, jq, file, unzip, make, gcc, g++, libtool, ca-certificates
- Compilation libraries: pkg-config, libffi-dev, libssl-dev, zlib1g-dev, libbz2-dev, libreadline-dev, libsqlite3-dev, libncurses5-dev, libxml2-dev, libxmlsec1-dev, liblzma-dev

### 3. Pterodactyl/Pelican Eggs
Complete egg configuration files that leverage the dual-container architecture:
- **Installation Phase**: Uses `python_uv_installer` (root, all dependencies)
- **Runtime Phase**: Uses version-specific images (non-root, secure)

Available eggs:
- **Pelican Panel** (PLCN_v3): [egg-python-uv.json](egg/egg-python-uv.json)
- **Pterodactyl Panel** (PTDL_v2): [egg-pterodactyl-python-uv.json](egg/egg-pterodactyl-python-uv.json)

### 4. Automated Egg Generation (hen.py)
Intelligent automation system that generates eggs from templates and scripts:
- **Templates**: [script/hens/](script/hens/) - Egg templates for both Pelican and Pterodactyl formats
- **Scripts**: [script/install.sh](script/install.sh), [script/start.sh](script/start.sh) - Installation and startup logic
- **Generator**: [script/hen.py](script/hen.py) - Automatically injects scripts into templates with format-specific escaping
- **Output**: [egg/](egg/) - Final generated eggs ready for distribution

The generator handles format differences automatically:
- Pelican: Standard JSON escaping, `startup_commands.Default` field
- Pterodactyl: `\r\n` newlines, backslash escaping, `startup` field

## Features

### Core Features
- **UV Package Manager**: Lightning-fast Python package installer and resolver, written in Rust (10-100x faster than pip)
- **Dual Installation Modes**:
  - **pip mode**: `uv pip install` with requirements.txt (traditional workflow)
  - **uv mode**: `uv sync` with pyproject.toml + uv.lock (modern project management)
- **Multi-Version Support**: Python versions from 3.8 to 3.14
- **Multi-Architecture**: Available for both `linux/amd64` and `linux/arm64`

### Security & Reliability
- **Dual-Container Architecture**: 
  - Installation uses root container with all dependencies
  - Runtime uses non-root container for enhanced security
- **No Permission Issues**: Pre-installed build dependencies eliminate `apt` permission errors
- **Isolated Environments**: Each project runs in its own container with proper isolation

### Developer Experience
- **Auto-Updates**: Built-in support for git repository auto-updates on startup
- **Flexible Dependency Management**: Support for requirements.txt, pyproject.toml, or direct package installation
- **Zero Manual Maintenance**: Automated egg generation via hen.py keeps configurations in sync
- **CI/CD Ready**: GitHub Actions workflows automatically build images and generate eggs

## Why UV?

[UV](https://github.com/astral-sh/uv) is a drop-in replacement for pip that offers:
- 10-100x faster package installation
- Better dependency resolution
- Improved caching mechanisms
- Written in Rust for maximum performance

## Configuration Options

The eggs support comprehensive configuration through environment variables:

### Core Settings
- **GIT_ADDRESS**: Your git repository URL (e.g., `https://github.com/username/repo`)
- **BRANCH**: Specific branch to clone (optional, defaults to repo's default branch)
- **USERNAME/ACCESS_TOKEN**: Git authentication credentials (for private repos)
- **AUTO_UPDATE**: Enable automatic git pull on startup (`0` = disabled, `1` = enabled)
- **USER_UPLOAD**: Skip git cloning if you're uploading files manually (`0` = false, `1` = true)

### Python Configuration
- **PY_FILE**: Main Python file to execute (default: `app.py`)
- **DEPENDENCY_INSTALL_MODE**: Choose dependency installation method
  - `pip` (default): Uses `uv pip install` with requirements.txt + additional packages
  - `uv`: Uses `uv sync` with pyproject.toml (ignores requirements.txt and PY_PACKAGES)

### Package Management (pip mode only)
- **REQUIREMENTS_FILE**: Requirements file name (default: `requirements.txt`)
- **PY_PACKAGES**: Additional packages to install (space-separated, e.g., `flask requests`)

### UV Mode (when DEPENDENCY_INSTALL_MODE=uv)
When using uv mode, dependencies are managed through:
- **pyproject.toml**: Project configuration and direct dependencies
- **uv.lock**: Locked dependency versions (auto-generated by uv)
- Environment: `/home/container/.local/uv` (set via UV_PROJECT_ENVIRONMENT)


## Available Images

All images are hosted on GitHub Container Registry (ghcr.io) and built for both amd64 and arm64 architectures.

### Runtime Images (Python with UV)

* [`python_uv_3.14`](/python_uv/3.14) - Python 3.14 with UV
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.14`
* [`python_uv_3.13`](/python_uv/3.13) - Python 3.13 with UV  
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.13`
* [`python_uv_3.12`](/python_uv/3.12) - Python 3.12 with UV
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.12`
* [`python_uv_3.11`](/python_uv/3.11) - Python 3.11 with UV
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.11`
* [`python_uv_3.10`](/python_uv/3.10) - Python 3.10 with UV
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.10`
* [`python_uv_3.9`](/python_uv/3.9) - Python 3.9 with UV
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.9`
* [`python_uv_3.8`](/python_uv/3.8) - Python 3.8 with UV
  * `ghcr.io/dong-chen-1031/yolks:python_uv_3.8`

### Installer Image

* [`python_uv_installer`](/python_uv/installer) - Dedicated installation environment
  * `ghcr.io/dong-chen-1031/yolks:python_uv_installer`
  * Based on Python 3.13 with all build dependencies
  * Runs as root for package compilation
  * Used during egg installation phase only

## Installation & Startup Flow

### Installation Phase (First-Time Setup)

The installation script runs in the `python_uv_installer` container with root privileges:

1. **Repository Cloning**
   - Clones your git repository to `/mnt/server`
   - Supports authentication via USERNAME/ACCESS_TOKEN
   - Handles branch selection

2. **Dependency Installation** (based on DEPENDENCY_INSTALL_MODE)
   
   **pip mode** (traditional):
   ```bash
   uv pip install -U --prefix .local ${PY_PACKAGES}
   uv pip install -U --prefix .local -r requirements.txt
   ```
   
   **uv mode** (modern):
   ```bash
   uv sync  # Uses pyproject.toml + uv.lock
   ```

### Runtime Phase (Every Startup)

The startup script runs in the version-specific runtime container as non-root user:

1. **Auto-Update** (if enabled)
   - Pulls latest changes from git repository
   
2. **Application Launch** (based on DEPENDENCY_INSTALL_MODE)
   
   **pip mode**:
   ```bash
   export PATH=/home/container/.local/bin:$PATH
   export PYTHONPATH=/home/container/.local
   python ${PY_FILE}
   ```
   
   **uv mode**:
   ```bash
   export UV_PROJECT_ENVIRONMENT="/home/container/.local/uv"
   uv run ${PY_FILE}
   ```

### Why Dual Containers?

- **Installation**: Needs root access to compile packages with C extensions (cryptography, lxml, etc.)
- **Runtime**: Runs as non-root for security (Pterodactyl/Pelican best practice)
- **Separation of Concerns**: Build dependencies don't bloat runtime images

## Building & Development

### Automated Builds

Images are automatically built via GitHub Actions workflows:

#### Runtime Images ([.github/workflows/python.yml](.github/workflows/python.yml))
- **Triggers**: Push to `python_uv/3.*/**`, weekly schedule (Monday), manual dispatch
- **Output**: `ghcr.io/dong-chen-1031/yolks:python_uv_3.{8-14}`
- **Architectures**: linux/amd64, linux/arm64

#### Installer Image ([.github/workflows/installer.yml](.github/workflows/installer.yml))
- **Triggers**: Push to `python_uv/installer/**`, weekly schedule (Monday), manual dispatch
- **Output**: `ghcr.io/dong-chen-1031/yolks:python_uv_installer`
- **Architectures**: linux/amd64, linux/arm64

#### Egg Generation ([.github/workflows/hen.yml](.github/workflows/hen.yml))
- **Triggers**: Push to `script/install.sh`, `script/start.sh`, `script/hens/**`, `script/hen.py`, manual dispatch
- **Process**: Runs `hen.py` to regenerate eggs from templates
- **Output**: Auto-commits updated eggs to `egg/` directory

### Local Building

Build runtime images:
```bash
docker build -t python_uv:3.13 ./python_uv/3.13
```

Build installer image:
```bash
docker build -t python_uv_installer ./python_uv/installer
```

Generate eggs locally:
```bash
cd script
python3 hen.py
```

### Development Workflow

1. **Modify Scripts**: Edit `script/install.sh` or `script/start.sh`
2. **Update Templates**: Modify egg templates in `script/hens/` if needed
3. **Test Locally**: Run `python3 hen.py` to verify generation
4. **Push Changes**: GitHub Actions automatically regenerates and commits eggs
5. **No Manual Egg Editing**: Never edit files in `egg/` directly - they're auto-generated!

## Project Structure

```
yolks-uv/
├── .github/workflows/
│   ├── python.yml          # Builds runtime images (3.8-3.14)
│   ├── installer.yml       # Builds installer image
│   └── hen.yml            # Auto-generates eggs from templates
├── python_uv/
│   ├── 3.8/ ... 3.14/     # Runtime image Dockerfiles
│   └── installer/         # Installer image Dockerfile
├── script/
│   ├── hens/              # Egg templates (source of truth)
│   │   ├── egg-python-uv.json              # Pelican template
│   │   └── egg-pterodactyl-python-uv.json  # Pterodactyl template
│   ├── hen.py             # Egg generator (the mother hen 🐔)
│   ├── install.sh         # Installation script
│   └── start.sh           # Startup script
├── egg/                   # Generated eggs (DO NOT EDIT)
│   ├── egg-python-uv.json              # Auto-generated Pelican egg
│   └── egg-pterodactyl-python-uv.json  # Auto-generated Pterodactyl egg
├── LICENSE.md
└── README.md
```

### Key Concepts

- **Templates (hens/)**: Source of truth for egg metadata, variables, and structure
- **Scripts (*.sh)**: Reusable installation and startup logic
- **Generator (hen.py)**: Injects scripts into templates with proper escaping
- **Output (egg/)**: Final eggs ready for import (auto-generated, never edited manually)

The "hen" metaphor: A mother hen (hen.py) lays eggs from templates and scripts 🐔→🥚

## Contributing

Contributions are welcome! Here's how to add features:

### Adding a New Python Version

1. Create directory: `python_uv/3.XX/`
2. Add Dockerfile based on existing versions
3. Update `.github/workflows/python.yml` matrix:
   ```yaml
   python-version: ['3.8', '3.9', '3.10', '3.11', '3.12', '3.13', '3.14', '3.XX']
   ```
4. Update egg templates in `script/hens/` to include new version in `docker_images`
5. Run `hen.py` to regenerate eggs
6. Update README with new version

### Modifying Installation/Startup Logic

1. Edit `script/install.sh` or `script/start.sh`
2. Test changes locally
3. Run `python3 script/hen.py` to regenerate eggs
4. Verify generated eggs in `egg/` directory
5. Push changes - GitHub Actions will auto-update eggs

### Adding Build Dependencies

Edit `python_uv/installer/Dockerfile` and add packages to the `apt-get install` command.

### Updating Egg Variables

Edit templates in `script/hens/`:
- `egg-python-uv.json` for Pelican (PLCN_v3 format)
- `egg-pterodactyl-python-uv.json` for Pterodactyl (PTDL_v2 format)

Then regenerate with `hen.py`.

## Troubleshooting

### Common Issues

#### Permission Denied During Installation
**Solution**: Ensure you're using the latest egg which uses `python_uv_installer` as the installation container. This container runs as root and has all necessary permissions.

#### Packages Fail to Compile
**Solution**: Check if the required compilation libraries are in `python_uv/installer/Dockerfile`. Common missing libraries can be added to the apt-get install command.

#### UV Mode Not Working
**Solution**: Ensure you have both `pyproject.toml` and `uv.lock` in your repository. Set `DEPENDENCY_INSTALL_MODE=uv` in the egg configuration.

#### Auto-Update Not Pulling Latest Changes
**Solution**: Set `AUTO_UPDATE=1` and ensure your git repository is accessible. For private repos, configure USERNAME and ACCESS_TOKEN.

#### Generated Eggs Don't Match Templates
**Solution**: Run `python3 script/hen.py` locally to regenerate. Never edit files in `egg/` directly - they're overwritten by automation.

## FAQ

**Q: What's the difference between pip mode and uv mode?**
- **pip mode**: Traditional workflow with requirements.txt, compatible with existing projects
- **uv mode**: Modern approach using pyproject.toml + uv.lock for reproducible builds

**Q: Why separate installer and runtime images?**
- Security: Runtime runs as non-root user
- Efficiency: Build tools don't bloat runtime images  
- Reliability: Root access during installation prevents permission errors

**Q: Can I use this with private GitHub repositories?**
Yes! Set the USERNAME and ACCESS_TOKEN variables in your egg configuration.

**Q: Why is it called "hen.py"?**
A mother hen lays eggs! 🐔 The generator (hen.py) creates eggs from templates (hens/) and scripts.

**Q: Do I need to manually update eggs after changing scripts?**
No! GitHub Actions automatically runs hen.py and commits updated eggs when scripts change.

**Q: Which Python version should I use?**
Use the latest stable version (3.13) unless you have specific compatibility requirements. All versions 3.8-3.14 are supported.

## License

See [LICENSE.md](LICENSE.md) for details.

## Credits

- **Docker Images**: Based on [Pelican Eggs Yolks](https://github.com/pelican-eggs/yolks) with UV integration
- **Egg Configuration**: Based on [Pelican Eggs Generic](https://github.com/pelican-eggs/generic) with enhanced UV support
- **UV Package Manager**: [astral-sh/uv](https://github.com/astral-sh/uv) - An extremely fast Python package installer and resolver

## Links

- **Repository**: [Dong-Chen-1031/yolks-uv](https://github.com/Dong-Chen-1031/yolks-uv)
- **Docker Images**: [ghcr.io/dong-chen-1031/yolks](https://github.com/Dong-Chen-1031/yolks-uv/pkgs/container/yolks)
- **Upstream Projects**:
  - [Pelican Eggs](https://github.com/pelican-eggs)
  - [UV Package Manager](https://github.com/astral-sh/uv)
  - [Pterodactyl Panel](https://pterodactyl.io/)

---

**Made with 🐔 by the hen.py automation system**