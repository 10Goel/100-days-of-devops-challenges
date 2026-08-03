# Commands Used

## Open current user's crontab

```bash
crontab -e
```

---

## View scheduled cron jobs

```bash
crontab -l
```

---

## Remove all cron jobs

```bash
crontab -r
```

---

## Remove with confirmation

```bash
crontab -i -r
```

---

## Edit another user's crontab (Root)

```bash
sudo crontab -u username -e
```

---

## View another user's cron jobs

```bash
sudo crontab -u username -l
```

---

## Cron Time Format

```text
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of Week (0-7)
│ │ │ └──── Month
│ │ └────── Day of Month
│ └──────── Hour
└────────── Minute
```

---

## Every minute

```bash
* * * * * command
```

---

## Every 5 minutes

```bash
*/5 * * * * command
```

---

## Every hour

```bash
0 * * * * command
```

---

## Every day at midnight

```bash
0 0 * * * command
```

---

## Every Sunday at 2 AM

```bash
0 2 * * 0 command
```

---

## Every Monday at 6 PM

```bash
0 18 * * 1 command
```

---

## Every 1st day of every month

```bash
0 0 1 * * command
```

---

## Run a shell script every day

```bash
0 9 * * * /home/user/script.sh
```

---

## Save and Exit (Vim)

```
Esc
:wq
```

---

## Exit Without Saving

```
Esc
:q!
```

---

## Verify Cron Service

Ubuntu

```bash
systemctl status cron
```

RHEL/CentOS

```bash
systemctl status crond
```

---

## Start Cron Service

Ubuntu

```bash
sudo systemctl start cron
```

CentOS

```bash
sudo systemctl start crond
```

---

## Enable Cron at Boot

Ubuntu

```bash
sudo systemctl enable cron
```

CentOS

```bash
sudo systemctl enable crond
```

---

## View Cron Logs

Ubuntu

```bash
grep CRON /var/log/syslog
```

CentOS

```bash
cat /var/log/cron
```
