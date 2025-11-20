## ⚙️ WSL2 VPN DNS Fix Manual

This guide helps you resolve network connectivity issues (specifically DNS resolution failures) in WSL2 when connected to a restrictive corporate VPN.

### 1\. 🔍 Diagnosis: Confirming the DNS Issue

Before fixing, you must confirm that the problem is a **DNS timeout** (VPN blocking public DNS) and not a full network block.

| Command | Location | Purpose | Expected Result (Failure) |
| :--- | :--- | :--- | :--- |
| `ping 8.8.8.8 -c 4` | WSL Terminal | Tests basic network path to the internet. | **Success** (0% loss) $\rightarrow$ Basic network is fine. |
| `dig archive.ubuntu.com` | WSL Terminal | Tests DNS resolution using current settings. | **Failure** (Timed out/No servers reached) $\rightarrow$ Confirms DNS issue. |

-----

### 2\. 🎣 Recovery: Finding the Corporate DNS IP

Your corporate VPN only allows DNS queries to its internal servers. You must recover these IPs from your Windows host.

| Step | Command | Location |
| :--- | :--- | :--- |
| 1. **Run IP Config** | `ipconfig /all` | **Windows PowerShell** or **CMD** (while connected to VPN) |
| 2. **Locate Adapter** | Scroll to the network adapter used by your VPN (e.g., "PPP Adapter," "Ethernet adapter VPN," or adapter name matching your VPN client). |
| 3. **Record IPs** | Look for the **`DNS Servers . . . . . . . . . . . :`** line. **Record ALL the IP addresses** listed here (e.g., `10.x.x.x`, `172.x.x.x`). |

-----

### 3\. 🛠️ Fix: Applying Static DNS Configuration

To prevent WSL from using its broken dynamic DNS settings, we force it to use the corporate IPs and disable the automatic file generation.

#### A. Disable Automatic `/etc/resolv.conf` Generation

1.  Edit the WSL system configuration file:
    ```bash
    sudo nano /etc/wsl.conf
    ```
2.  Ensure these lines are present to prevent WSL from overwriting your custom configuration:
    ```ini
    [network]
    generateResolvConf = false
    ```
3.  Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

#### B. Configure the Corporate DNS Servers

1.  **Remove** the existing dynamic `/etc/resolv.conf` file (it's often a symlink):
    ```bash
    sudo rm -f /etc/resolv.conf
    ```
2.  **Create** a new static file:
    ```bash
    sudo nano /etc/resolv.conf
    ```
3.  Add the corporate IP addresses you recovered from PowerShell, listing them one per line in order of preference (primary first):
    ```conf
    nameserver <Corporate DNS IP 1>
    nameserver <Corporate DNS IP 2>
    nameserver <Corporate DNS IP 3>
    ```
4.  Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

#### C. Apply Changes

1.  **Shut down** the WSL virtual machine from Windows PowerShell:
    ```powershell
    wsl --shutdown
    ```
2.  Restart your Ubuntu distribution.

-----

### 4\. ✅ Verification

Run your update command to confirm success:

```bash
sudo apt update
```

If successful, the output will show the package lists being downloaded.

-----

If you encounter a **`certificate verification failed`** or **SSL/TLS error** after fixing DNS, you will need to install your custom corporate CA certificates into the Ubuntu trust store.
