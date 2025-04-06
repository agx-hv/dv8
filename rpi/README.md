# Cross-compiling ROS Binaries for Raspberry Pi 2 Zero W

This guide walks you through the process of cross-compiling ROS Noetic binaries for the Raspberry Pi Zero 2 W, which uses an ARM64 processor. Rather than compiling directly on the Pi — which is slow and resource-limited — this approach allows you to build binaries on a fast x86-64 machine and run them immediately on a fresh install of Raspberry Pi OS (Bookworm, 64-bit).

⚠️ Note: ROS Noetic is only officially supported on Ubuntu 20.04 (Focal), which reaches end-of-Life (EOL) in May 2025. It is strongly recommended to use Ubuntu 20.04 in a chroot, VM, or container for building, rather than on your host system.

---
###🚀 Why Cross-Compile?
Cross-compiling provides more than just faster build times. Here are some major advantages:

✅ No Need to Install ROS on the Pi
- The Raspberry Pi does not need ROS Noetic installed.
- This keeps the Pi system clean, reduces maintenance, and avoids dependency issues.

✅ Compatible with the Latest Raspberry Pi OS
- The generated binaries run perfectly on Raspberry Pi OS Bookworm (64-bit).
- No need to downgrade to deprecated or unsupported distributions like Ubuntu 20.04 Focal.

✅ Faster Builds and Less Heat
- Builds are done on your powerful dev machine, not the Pi.
- No prolonged compilation times that could overheat or wear down the Pi.

✅ Minimal Footprint
- Deployment is as simple as copying over the compiled binary.
- No need for dev tools, compilers, or ROS packages on the Pi.

✅ Predictable, Reproducible Results
- Build environments are fully controlled and reproducible using Docker or chroot.
- Binaries built this way can be versioned, tested in CI, and rolled out reliably across multiple Pi devices.

### ⚙️ Why Use Zig as the Linker?
Traditional cross-compilation often requires setting up:

- A cross toolchain (e.g., gcc-aarch64-linux-gnu)
- A compatible sysroot and linker
- Manual configuration of compiler flags and environment variables

Zig replaces all of that with a single, portable, zero-setup linker.

🔧 Zig Makes Cross-Compilation Simple:
- Automatically configures the correct linker and system libraries for the target.
- No need to mess with LD_LIBRARY_PATH, sysroots, or architecture-specific GCC setups.
- Works seamlessly with Rust via cargo-zigbuild.

✅ Cross-Compiles Rust to ARM64 Without Hassle:
- Just run cargo zigbuild --target aarch64-unknown-linux-gnu and you're done.
- No need to install or configure complex toolchains.
- The resulting binary will link correctly against the Pi’s system libraries out-of-the-box.

###🦀 Why Use Rust?
Rust is a modern systems programming language designed for performance, reliability, and safety — all of which make it an excellent fit for developing ROS nodes on embedded platforms like the Raspberry Pi Zero 2 W.

Here’s why Rust is a great choice for this cross-compilation workflow:

✅ Memory Safety Without Runtime Overhead
Rust guarantees memory safety at compile time, without relying on a garbage collector. This eliminates entire classes of bugs — such as null pointer dereferences, buffer overflows, and use-after-free — while still delivering performance close to C/C++. This is especially important on embedded devices where resources are limited, and bugs can be hard to debug.

✅ Lightweight, High-Performance Binaries
Rust compiles directly to efficient native code with zero-cost abstractions. The resulting binaries are small, fast, and ideal for running on the Pi Zero 2 W, which has limited CPU and memory resources.

✅ Safe Concurrency by Design
Rust’s ownership system also applies to concurrent programming. It makes data races impossible by default, allowing you to confidently build multithreaded ROS nodes that handle multiple topics, services, or sensors in parallel — without risking undefined behavior or hard-to-find bugs.

✅ Writing ROS Nodes in Rust
With libraries like rosrust, you can write fully-functional ROS Noetic nodes in pure Rust — supporting publishers, subscribers, services, and parameters. This brings all of Rust’s safety and performance benefits into the ROS ecosystem, without needing to fall back on C++.


### System Requirements and Prerequisites

- x86-64 Development Machine
- Ubuntu 20.04 LTS (in VM, Docker container, or chroot)
- Git

### Installation Steps

1. **Install ROS Noetic on Ubuntu 20.04 LTS**
    ```bash
    sudo apt update && sudo apt install curl gnupg -y
    sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
    sudo apt update && sudo apt install ros-noetic-ros-base -y
    
2. **Install Rust**
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

3. **Install aarch64 toolchain for Linux**
    ```bash
    rustup target add aarch64-unknown-linux-gnu 

4. **Download and extract Zig 0.9.0 compiler for x64**
    ```bash
    curl -s -L https://ziglang.org/download/0.9.0/zig-linux-x86_64-0.9.0.tar.xz | tar xvJ -C ~

5. **Automatically add zig binary to PATH environment variable**
    ```bash
    echo "export PATH=$PATH:$HOME/zig-linux-x86_64-0.9.0/" >> $HOME/.bashrc && source $HOME/.bashrc

6. **Test Zig compiler path works and correctly displays its version**
    ```bash
    zig version

7. **Install cargo-zigbuild to allow Rust to use zig as a linker when cross-compiling**
    ```bash
    cargo install --locked cargo-zigbuild 

8. **Clone this repository and change directory into the rosbin project directory**
    ```bash
    git clone https://github.com/agx-hv/dv8 && cd dv8/rpi/rosbin

9. **Source ros environment and set the ROSRUST_MSG_PATH environment variable**
    ```bash
    source /opt/ros/noetic/setup.bash && export ROSRUST_MSG_PATH=/path/to/directory/containing/bumperbot_controller

10. **Build the arm64 binary**
    ```bash
    cargo zigbuild --release --target aarch64-unknown-linux-gnu

11. **Copy the binary over to the Raspberry Pi**
    ```bash
    scp target/aarch64-unknown-linux-gnu/release/rosbin dv8@<RASPI_IP_ADDRESS>:~/


