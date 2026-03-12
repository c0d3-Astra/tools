# tmux-gdb-dashboard

A lightweight **tmux automation script** that launches **GDB with GDB Dashboard** in a structured multi-pane layout.

The script automatically creates a **2×2 tmux debugging workspace**, starts **gdb**, and routes different dashboard modules (source, registers, memory) to dedicated panes.

This setup is especially useful for:

* Binary exploitation
* Reverse engineering
* Low-level debugging
* CTF challenges (e.g., picoCTF, pwn challenges)

---

# Features

* Automatic **tmux session creation**
* Generates a **2×2 debugging grid**
* Runs **GDB automatically**
* Integrates with **GDB Dashboard**
* Sends dashboard modules to **separate panes**
* Automatically selects the correct **stack register (`$esp` / `$rsp`)**
* Requires **zero manual pane configuration**

---

# Layout

```
+----------------------+----------------------+
|        SOURCE        |      REGISTERS       |
|                      |                      |
|                      |                      |
+----------------------+----------------------+
|       GDB CONSOLE    |        MEMORY        |
|      (interactive)   |       (stack)        |
+----------------------+----------------------+
```

Pane description:

| Pane         | Purpose                     |
| ------------ | --------------------------- |
| Top Left     | Displays **source code**    |
| Top Right    | Displays **registers**      |
| Bottom Left  | **Interactive GDB console** |
| Bottom Right | **Stack memory viewer**     |

---

# Requirements

Install the following dependencies:

```
tmux
gdb
gdb-dashboard
```

## Install tmux

```
sudo apt install tmux
```

## Install gdb

```
sudo apt install gdb
```

## Install GDB Dashboard

```
git clone https://github.com/cyrus-and/gdb-dashboard.git
echo "source ~/gdb-dashboard/.gdbinit" >> ~/.gdbinit
```

---

# Installation

Clone or download the script.

```
git clone https://github.com/c0d3-Astra/tools.git
cd tools
```

Make it executable:

```
chmod +x tm4.sh
```

---

# Usage

```
./tm4.sh <binary> <arch>
```

### Example

64-bit binary

```
./tm4.sh ./vuln 64
```

32-bit binary

```
./tm4.sh ./vuln 32
```

---

# Architecture Handling

The script automatically chooses the correct stack pointer register.

| Architecture | Stack Register |
| ------------ | -------------- |
| 32-bit       | `$esp`         |
| 64-bit       | `$rsp`         |

Example logic used:

```
if [ "$ARCH" = "32" ]; then
    STACK_REG="$esp"
else
    STACK_REG="$rsp"
fi
```

---

# What the Script Does

### 1. Creates a tmux session

```
tmux new-session -d -s gbd_tmux
```

### 2. Builds a 2×2 pane grid

```
tmux split-window -h
tmux split-window -v
tmux split-window -v
```

### 3. Extracts pane TTYs

Each pane receives its own **TTY**, which allows GDB dashboard modules to print into specific panes.

```
tmux display-message -p "#{pane_tty}"
```

### 4. Launches GDB

```
gdb <binary>
```

### 5. Configures Dashboard Modules

```
dashboard -layout source registers memory assembly breakpoints variables
```

### 6. Redirects outputs

```
dashboard source -output <tty>
dashboard registers -output <tty>
dashboard memory -output <tty>
```

### 7. Watches the stack

```
dashboard memory watch $rsp 256
```

(or `$esp` for 32-bit)

---

# Example Debugging Workflow

Run the script:

```
./gdb_tmux.sh ./vuln 64
```

Inside GDB:

```
break main
run
next
step
x/20gx $rsp
```

You will see:

* **source updating live**
* **registers updating automatically**
* **stack memory updating**
* **interactive console for commands**

---

# Advantages

Compared to normal GDB:

| Normal GDB              | tmux-gdb-dashboard       |
| ----------------------- | ------------------------ |
| Single terminal         | Multi-pane view          |
| Hard to track registers | Registers always visible |
| Manual memory checks    | Auto stack watch         |
| Messy output            | Clean separation         |

---

# Use Cases

This setup is great for:

* **CTF binary exploitation**
* **Kernel debugging**
* **Assembly learning**
* **Stack analysis**
* **Exploit development**

---

# Troubleshooting

## Dashboard not loading

Make sure `.gdbinit` contains:

```
source ~/gdb-dashboard/.gdbinit
```

## tmux session already exists

Kill the previous session:

```
tmux kill-session -t gbd_tmux
```

## Binary not loading

Ensure the binary is executable:

```
chmod +x ./binary
```

---

# Future Improvements

Potential upgrades:

* Automatic architecture detection
* pwndbg / gef compatibility
* dynamic pane resizing
* extra panes for stack / heap
* breakpoint pane
* syscall tracing

---

# License

MIT License

---

# Author

Created for **binary exploitation and debugging workflows**.
