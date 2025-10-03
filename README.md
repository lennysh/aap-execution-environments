# Ansible Execution Environment (EE) Builder

This repository contains Ansible automation to build custom Execution Environments (EEs) for use with Ansible Automation Platform (or ansible-navigator). It uses `ansible-builder` to assemble the EEs based on definition files and pushes them to a container registry.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
  - [Ansible Vault](#ansible-vault)
- [Usage](#usage)
  - [Building an Execution Environment](#building-an-execution-environment)
- [Available Execution Environments](#available-execution-environments)
- [How It Works](#how-it-works)
- [License](#license)

## Prerequisites

Before you begin, ensure you have the following tools installed on your system:

-   `ansible-core`
-   `ansible-builder`
-   `podman` (or another compatible container engine)
-   `ansible-vault`

You will also need one or more Ansible collections:
```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Configuration

### Ansible Vault

This project uses Ansible Vault to manage sensitive registry credentials.

1.  **Create the Vault File:**
    Create a file named `.vault.yml` in the root of the repository with the following content:

    ```yml
    ---
    # Credentials for the source registry (e.g., registry.redhat.io)
    registry_name: "registry.redhat.io"
    registry_user: "" # Your Red Hat username/service account
    registry_pass: "" # Your Red Hat password/service account token
    registry_token: "" # Your Red Hat Offline Token for Ansible Galaxy

    # Credentials for the destination registry (e.g., quay.io)
    dest_registry_name: "quay.io"
    dest_registry_user: "" # Your destination registry username
    dest_registry_pass: "" # Your destination registry password/token
    ```

2.  **Create a Vault Password File:**
    The `ansible.cfg` in this repository is configured to use a vault password file named `.vault_pass.txt`. Create this file and add your vault password to it.

    ```bash
    echo "your_super_secret_vault_password" > .vault_pass.txt
    ```

3.  **Encrypt the Vault File:**
    Encrypt the `.vault.yml` file using `ansible-vault`.

    ```bash
    ansible-vault encrypt .vault.yml --vault-password-file .vault_pass.txt
    ```

## Usage

### Building an Execution Environment

To build an EE, run the `build_ee.yml` playbook and specify which EE definition file you want to use with the `ee_vars_file` extra variable.

**Example:**

To build the `ee-casc-rhel9` environment for Ansible Automation Platform 2.6, run the following command:

```bash
ansible-playbook build_ee.yml --extra-vars "ee_vars_file=vars/ansible-automation-platform-26/ee-casc-rhel9.yml"
```

The playbook will build the container image and push it to the destination registry specified in your `.vault.yml` file (e.g., `quay.io/lshirley/ansible-automation-platform-26/ee-casc-rhel9`).

## Available Execution Environments

The definitions for the execution environments are located in the `vars/` directory.

| Purpose | Ansible Core | OS | Var File | Public Repo |
| :------ | :----------: |:-: | :------: | :---------: |
| Config as Code for AAP 2.4 | 2.16.x | RHEL 8 | [Link](vars/ansible-automation-platform-24/ee-casc-rhel8.yml) | [Link](https://quay.io/repository/lshirley/ansible-automation-platform-24/ee-casc-rhel8) |
| Config as Code for AAP 2.4 | 2.16.x | RHEL 9 | [Link](vars/ansible-automation-platform-24/ee-casc-rhel9.yml) | [Link](https://quay.io/repository/lshirley/ansible-automation-platform-24/ee-casc-rhel9) |
| Config as Code for AAP 2.5 | 2.16.x | RHEL 8 | [Link](vars/ansible-automation-platform-25/ee-casc-rhel8.yml) | [Link](https://quay.io/repository/lshirley/ansible-automation-platform-25/ee-casc-rhel8) |
| Config as Code for AAP 2.5 | 2.16.x | RHEL 9 | [Link](vars/ansible-automation-platform-25/ee-casc-rhel9.yml) | [Link](https://quay.io/repository/lshirley/ansible-automation-platform-25/ee-casc-rhel9) |
| Config as Code for AAP 2.6 | 2.16.x | RHEL 9 | [Link](vars/ansible-automation-platform-26/ee-casc-rhel9.yml) | [Link](https://quay.io/repository/lshirley/ansible-automation-platform-26/ee-casc-rhel9) |

## How It Works

1.  The `build_ee.yml` playbook is the main entry point.
2.  It includes the variable file specified by `ee_vars_file` to get the specific EE definition.
3.  It calls the `build_ee` role.
4.  The role creates a `_build` directory and templates out the necessary files for `ansible-builder`, including `execution-environment.yml`, `requirements.txt`, etc.
5.  It runs the `ansible-builder build` command to create the container image.
6.  Finally, it tags the new image with `latest` and a timestamp, and pushes both tags to the destination registry.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.