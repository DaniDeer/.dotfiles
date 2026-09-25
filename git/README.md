# Git module

Git preferences, identity bootstrap, and optional GitHub CLI authentication.

## Install

Run the root installer from the dotfiles repository:

```sh
GIT_USER_NAME='Your Name' \
GIT_USER_EMAIL='you@example.com' \
./install
```

Identity variables are optional when Git identity is already configured. The module is safe to rerun and does not store credentials or signing keys in the repository.

## Configuration layout

The installer builds one self-contained `~/.gitconfig` from `git/config` and the optional identity variables. It intentionally avoids Git config includes to keep the setup simple and predictable, especially in devcontainers where host-specific include paths or files may not exist.

## Proxy configuration

Git uses the host's `http_proxy`, `https_proxy`, and `no_proxy` environment variables directly. The installer does not persist proxy values, so devcontainers without these variables do not inherit host-specific settings.

## GitHub authentication

Preferred for Git operations: use an SSH key held by your local SSH agent and forward the agent into the VM. This avoids copying private keys or tokens into disposable environments.

For short-lived automation, inject a fine-grained GitHub token from a secret manager as `GH_TOKEN`. GitHub CLI reads it for the current process without requiring a dotfiles change:

```sh
GH_TOKEN="$GITHUB_TOKEN_FROM_SECRET_MANAGER" gh repo view
```

For a disposable VM, the module also supports explicit token bootstrap:

```sh
GH_TOKEN="$GITHUB_TOKEN_FROM_SECRET_MANAGER" ./install
```

This runs `gh auth login --with-token` and configures Git to use GitHub CLI credentials. GitHub CLI stores the credential using its configured credential store; check that store before using this on a persistent machine. Use a fine-grained token with the smallest required repository permissions and rotate or revoke it regularly. Never commit tokens, put them in this repository, or pass them in Terraform state.

## Provisioning

Run the root installer from Terraform cloud-init, an image bootstrap script, or a devcontainer setup step after cloning this repository:

```sh
GIT_USER_NAME="$YOUR_GIT_NAME" \
GIT_USER_EMAIL="$YOUR_GIT_EMAIL" \
/home/adminuser/prj/dotfiles/install
```

Use a secret manager for any credentials.