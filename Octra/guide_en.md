# Octra Node Guide
This guide provides step-by-step instructions for setting up Octra Node in. The process includes installing system dependencies, configuring the environment, and setting up the necessary variables.

-----------------------------------------------------------------

## Installation Steps
### 1. Install Required Dependencies
First, install the OCaml package finder, which is essential for managing OCaml packages.
```bash
sudo apt install -y ocaml-findlib
```

### 2. Set Up the Environment
Download and Execute Installation Script
```bash
wget https://raw.githubusercontent.com/BlockchainsHub/Testnet/refs/heads/main/Octra/env.sh
chmod +x env.sh
./env.sh
```

Configure Environment Variables
```bash
if ! grep -q "opam-init" "$HOME/.profile" 2>/dev/null; then
    echo '. "$HOME/.opam/opam-init/init.sh" > /dev/null 2>&1 || true' >> "$HOME/.profile"
fi
source "$HOME/.profile"
```

### 3. Install Required OCaml Packages
Install the necessary OCaml packages using OPAM.
```bash
opam install -y yojson cohttp-lwt-unix lwt_ppx
```

### 4. Configure and Run the Application
Download the configuration file.
```bash
wget https://raw.githubusercontent.com/BlockchainsHub/Testnet/refs/heads/main/Octra/config.ml
```

Compile and run the configuration.
```bash
ocamlfind ocamlopt -o config -thread -linkpkg -package yojson,cohttp-lwt-unix,unix,str,lwt_ppx config.ml
./config
```

> [!CAUTION]
> Make sure to backup your **`hash_id_config.json`** file!!! If you lose it, it will be impossible to authorize your node even with a private key.

![CleanShot 2025-01-02 at 10 08 53@2x](https://github.com/user-attachments/assets/84d90733-a135-4a5f-a8fc-4631a1ef3a31)

-----------------------------------------------------------------

<p align="center">
  &copy; 2025 BlockHub. All rights reserved.
</p>
