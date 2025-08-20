# Configuring Windows for Ansible using WinRM

This guide explains how to prepare a Windows machine for remote management with **Ansible** via **WinRM (Windows Remote Management)**.  
We will use the official Ansible script `ConfigureRemotingForAnsible.ps1` to make the required configuration changes.

---

## 1. Download the Configuration Script

The Ansible team provides a PowerShell script to automatically configure WinRM for Ansible.

📥 **Download it from the official repository:**

[ConfigureRemotingForAnsible.ps1 (GitHub link)](https://github.com/ansible/ansible/blob/stable-2.9/examples/scripts/ConfigureRemotingForAnsible.ps1)

Save the file somewhere on your Windows machine (e.g., `C:/Users/<YourUser>/Downloads`).

---

## 2. Open PowerShell as Administrator

1. Click **Start** → search for **PowerShell**.
2. Right–click **Windows PowerShell** and select **Run as administrator**.
3. Confirm the UAC prompt.

---

## 3. Run the Script

Before running the script, we will temporarily allow PowerShell scripts to run without restrictions in this session.

Replace `<path-to-script>` with the actual path to `ConfigureRemotingForAnsible.ps1`.

```powershell
$file = "C:/Path/To/ConfigureRemotingForAnsible.ps1"
powershell.exe -ExecutionPolicy Bypass -File $file -Verbose
```

<img width="652" height="170" alt="image" src="https://github.com/user-attachments/assets/7933045e-bc5c-4c1d-809e-4f2036ed9578" />

## 4. Expected Successful Output

If everything is configured correctly, you should see verbose output confirming that WinRM is enabled and ready for Ansible, similar to:

```plaintext
VERBOSE: Verifying WinRM service.
VERBOSE: PS Remoting is already enabled.
VERBOSE: SSL listener is already active.
VERBOSE: Basic auth is already enabled.
VERBOSE: Firewall rule already exists to allow WinRM HTTPS.
VERBOSE: HTTP: Enabled | HTTPS: Enabled
VERBOSE: PS Remoting has been successfully configured for Ansible.
```

---


## 5. What This Script Does

The script automatically:
- Enables **PowerShell Remoting**
- Configures **WinRM listeners** (both HTTP and HTTPS)
- Enables **Basic authentication**
- Adjusts the **Windows Firewall** to allow WinRM traffic
- Ensures compatibility with **Ansible’s WinRM connection plugin**

---

## 6. Next Steps in Ansible

In your Ansible inventory, specify that the Windows host uses WinRM, for example:

```ini
[windows]
winhost.example.com

[windows:vars]
ansible_user=Administrator
ansible_password=MySecurePassword
ansible_connection=winrm
ansible_port=5986
ansible_winrm_server_cert_validation=ignore
```
## Prerequisites (Control Node)

To connect Ansible to Windows hosts via WinRM, ensure your **Ansible control node** (the machine running Ansible) meets the following requirements:

- **Python 3.x** installed  
- **pip** (Python package manager)  
- Install the required Python packages:
  ```bash
  pip install pyopenssl
  pip install requests
  pip install pywinrm
  ```

- On Linux control nodes, you may also need development libraries for building Python packages (like `pyopenssl` or `cryptography`):
  | Distribution      | Package Name        |
  |-------------------|---------------------|
  | RHEL/CentOS/Fedora| libffi-devel, python-devel |
  | Debian/Ubuntu     | libffi-dev, python3-dev    |

> **Note:**  
> - `libffi-devel` and `python-devel` are **OS packages**, not Python packages.  
> - These are required mainly on Linux if you need to compile certain Python extensions.  
> - On Windows or macOS control nodes, you usually just need to install `pyopenssl`, `requests`, and `pywinrm` via `pip`.

---
## Troubleshooting: Network Profile is Public

**Problem:**  
When running `winrm quickconfig`, you may see a warning like:  

```
WinRM is not set up to allow remote access to this machine for management.
Network connection type is Public.
```

When the network profile is **Public**, Windows Firewall blocks WinRM ports (5985/5986) by default, and `quickconfig` cannot automatically create the required firewall rules.

---

**Solution:**  

1. **Check your current network profile:**
   ```powershell
   Get-NetConnectionProfile
   ```

   Example output:
   ```
   Name             : Ethernet
   NetworkCategory  : Public
   ```

2. **Change the network profile from Public to Private:**
   ```powershell
   Set-NetConnectionProfile -InterfaceAlias "Ethernet" -NetworkCategory Private
   ```

   > Replace `"Ethernet"` with the `InterfaceAlias` shown in your system (e.g., `"Ethernet 2"` or `"Wi-Fi"`).

3. **Verify the change:**
   ```powershell
   Get-NetConnectionProfile
   ```
   Should now show:
   ```
   NetworkCategory  : Private
   ```

4. **Re-run WinRM configuration:**
   ```powershell
   winrm quickconfig -q
   ```

   This time, WinRM will configure successfully and will add firewall rules for the correct network profile.

---

**Example:**
Before:
```
Name             : Ethernet
NetworkCategory  : Public
```
After:
```
Name             : Ethernet
NetworkCategory  : Private
```
Now WinRM should work with Ansible over the selected port.

---
**Tip:**  
If you run into authentication issues, make sure:
- The username and password are correct
- The firewall allows ports `5985` (HTTP) and `5986` (HTTPS)
- The script was run under **Administrator** privileges
  
✅ Now your Windows machine is ready to be managed by Ansible.
