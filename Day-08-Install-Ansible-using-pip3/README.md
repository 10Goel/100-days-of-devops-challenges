# Day 8 - Install Ansible using pip3 | KodeKloud 100 Days of DevOps Challenge

## 📌 Challenge Objective

Install **Ansible version 4.8.0** on the **Jump Host** using **pip3** and ensure that the Ansible binary is available globally so every user can execute Ansible commands.

---

## 📝 Problem Statement

The Nautilus DevOps team has decided to use **Ansible** as their Configuration Management tool.

The task required:

- Install Ansible version **4.8.0**
- Use **pip3** only (do not use yum/dnf)
- Ensure the Ansible executable is globally accessible
- Verify the installation

---

## 🛠️ Steps Performed

1. Connected to the Jump Host.
2. Verified Python and pip3 installation.
3. Installed Ansible version 4.8.0 using pip3.
4. Verified the installation using `ansible --version`.
5. Confirmed the binary location using `which ansible`.
6. Verified that the binary was globally accessible.

---

## ✅ Verification

Verified successfully using:

```bash
ansible --version
which ansible
```

Task completed successfully.

---

## 📚 Key Concepts Learned

- Configuration Management
- Infrastructure Automation
- Ansible
- pip3 package installation
- Python Package Index (PyPI)
- PATH Environment Variable
- Global executable binaries

---

## 🚀 Outcome

Successfully installed Ansible using pip3 and ensured that the Ansible executable was globally available for all users.
