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
ansible_winrm_transport=basic
ansible_port=5986
ansible_winrm_server_cert_validation=ignore
```

---

**Tip:**  
If you run into authentication issues, make sure:
- The username and password are correct
- The firewall allows ports `5985` (HTTP) and `5986` (HTTPS)
- The script was run under **Administrator** privileges

---

✅ Now your Windows machine is ready to be managed by Ansible.
