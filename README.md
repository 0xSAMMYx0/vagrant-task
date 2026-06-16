# Practical Task: Vagrant Deployment with Docker Provider

This repository contains the solution for the practical task to deploy a Java-based Spring Boot application in an isolated environment using Vagrant.

## Architecture & Troubleshooting (Apple Silicon / ARM64)

The original project configuration was designed for the `VirtualBox` provider using an `x86_64` Ubuntu box. When attempting to boot on Apple Silicon hardware (M-series Mac), it triggers a critical architecture incompatibility error (`platform architecture x86 is not supported on ARM`).

To resolve this issue and ensure native performance, the infrastructure was fully adapted to use the **Docker Provider**.

### Technical Breakdown of the Configuration Items:

* **`config.vm.provider "docker"`**
  Specifies Docker as the core virtualization engine instead of VirtualBox. This allows the environment to run natively on macOS without heavy x86 emulation overhead.
  
* **`d.image = "ubuntu:22.04"` & `"--platform", "linux/arm64"`**
  Pulls the official, lightweight Ubuntu image explicitly configured for the `arm64` architecture, matching the host machine's processor natively.

* **`d.has_ssh = false`**
  Disables the default SSH handshake layer. Since standard minimal Docker boxes do not run an SSH daemon by default, disabling this prevents Vagrant from hanging during the boot phase.

* **`d.entry_point = "bash"`**
  Overrides the container's standard entry point to use the Bash shell, allowing us to execute sequential provisioning commands directly.

* **`d.cmd = [...]` (Provisioning Pipeline)**
  Automates the complete environment setup sequentially upon container initialization:
  1. `apt-get update -y && apt-get install -y openjdk-17-jdk`: Updates package definitions and installs Java Development Kit (JDK 17) required by the application.
  2. `cd /vagrant`: Navigates to the synchronized project directory mapping the host code inside the container.
  3. `chmod +x gradlew && ./gradlew bootRun`: Grants execution permissions to the Gradle wrapper and builds/boots the Spring Boot application.
  4. `|| tail -f /dev/null`: An execution safety net that keeps the container alive for logs and troubleshooting if the build fails.

* **`config.vm.network "forwarded_port", guest: 8080, host: 8080`**
  Maps port 8080 from inside the container to port 8080 on the local host machine. This routes external local traffic straight to the embedded Netty server.

## How to Run

1. Make sure **Docker Desktop** is running on your machine.
2. Open your terminal in the repository root directory and execute:
   ```bash
   vagrant up --provider=docker
