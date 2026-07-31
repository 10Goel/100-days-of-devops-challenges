# Commands Used

## Connect to Application Servers

```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

---

## Become Root

```bash
sudo su -
```

---

## Edit SSH Configuration

```bash
nano /etc/ssh/sshd_config
```

---

## Disable Root Login

```text
PermitRootLogin no
```

---

## Restart SSH

```bash
systemctl restart sshd
```

---

## Verification

```bash
grep "^PermitRootLogin" /etc/ssh/sshd_config
```
