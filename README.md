# Dotfiles

Modular bootstrap configuration for disposable development environments.

## Install

From this repository:

```sh
GIT_USER_NAME="$YOUR_GIT_NAME" \
GIT_USER_EMAIL="$YOUR_GIT_EMAIL" \
./install
```

Root `install` runs each immediate module installer in lexical order. Add modules as directories containing an executable `install` script and module-specific files.

See [`git/README.md`](git/README.md) for Git identity, GitHub authentication, and provisioning details.

Use a secret manager for credentials. Do not put tokens, passwords, or private keys in this repository.

## Terraform cloud-init

Clone this repository during VM provisioning, then run its root installer as the target development user. The VM must already have access to the repository through a public URL, an SSH agent, or another bootstrap mechanism.

Example `cloud-init.yaml.tftpl`:

```yaml
# Terraform supplies dotfiles_repo, git_user_name, and git_user_email.
runcmd:
  - [sh, -c, "install -d -o vscode -g vscode /home/vscode/.dotfiles"]
  - [sh, -c, "runuser -u vscode -- git clone --depth=1 '${dotfiles_repo}' /home/vscode/.dotfiles"]
  - [sh, -c, "runuser -u vscode -- env GIT_USER_NAME='${git_user_name}' GIT_USER_EMAIL='${git_user_email}' /home/vscode/.dotfiles/install"]
```

Define repository and identity as Terraform variables:

```hcl
variable "dotfiles_repo" {
  type        = string
  description = "Clone URL for dotfiles repository"
}

variable "git_user_name" {
  type        = string
  description = "Git author and committer name"
}

variable "git_user_email" {
  type        = string
  description = "Git author and committer email"
}
```

Render the template with `templatefile` and pass the result as `user_data`:

```hcl
user_data = templatefile("${path.module}/cloud-init.yaml.tftpl", {
  dotfiles_repo = var.dotfiles_repo
  git_user_name = var.git_user_name
  git_user_email = var.git_user_email
})
```

The clone URL must already be accessible to the VM through a public repository, forwarded SSH agent, or another bootstrap mechanism. Keep tokens and private keys out of the template and Terraform state; retrieve them from a secret manager during bootstrap and expose `GH_TOKEN` only to the command that needs it.

## Possible extensions

Each module should own its installer, configuration, and focused documentation. Good candidates include:

| Module | Purpose |
| --- | --- |
| `shell` | Shell aliases, prompts, functions, and completion |
| `ssh` | Client defaults, host aliases, and agent integration; never private keys |
| `github` | GitHub CLI defaults and repository workflow helpers |
| `gpg` | Commit-signing configuration; keys remain external |
| `editor` | Vim, Neovim, or other editor configuration |
| `tmux` | Terminal multiplexer settings and session helpers |
| `direnv` | Project environment loading and approved environment hooks |
| `go` | Go environment, module proxy, and tool configuration |
| `python` | Python tools, package indexes, and virtual-environment helpers |
| `node` | Node version manager and package-manager settings |
| `containers` | Docker or Podman client configuration and aliases |
| `kubernetes` | `kubectl` helpers and context-aware shell functions |
| `cloud` | AWS, Azure, or GCP CLI defaults without credentials |
| `terraform` | Terraform CLI configuration and provider plugin cache |
| `devcontainer` | Shared development-container defaults and lifecycle helpers |
| `security` | Safe defaults, scanners, and local security tooling |

Modules should be idempotent, tolerate missing optional tools, and avoid network installs unless explicitly documented. Keep credentials, private keys, machine-specific paths, and generated state outside this repository.

Example module shape:

```text
module-name/
├── install
├── config/
└── README.md
```
