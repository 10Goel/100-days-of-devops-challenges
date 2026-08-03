# Linux Cron Jobs - Complete Notes

# What is Cron?

Cron is a Linux service that automatically executes commands or shell scripts at scheduled times.

Instead of running commands manually every day, Cron performs the task automatically in the background.

It is one of the oldest and most reliable scheduling systems available in Unix/Linux.

---

# What is Cron Daemon?

The Cron Daemon is a background service.

Its process name is:

cron

or

crond

depending on the Linux distribution.

It continuously checks all scheduled jobs every minute.

If the current time matches a scheduled job, Cron executes it automatically.

---

# What is Crontab?

Crontab stands for

CRON TABLE

It is simply a file containing scheduled jobs.

Every Linux user can have their own crontab.

Root also has a separate crontab.

---

# Crontab Commands

Edit

```bash
crontab -e
```

View

```bash
crontab -l
```

Delete

```bash
crontab -r
```

---

# Cron Time Format

```
* * * * * command
```

There are five time fields.

Minute

0-59

Hour

0-23

Day of Month

1-31

Month

1-12

Day of Week

0-7

Where

0 or 7 = Sunday

1 = Monday

2 = Tuesday

3 = Wednesday

4 = Thursday

5 = Friday

6 = Saturday

---

# Example

```
30 14 * * * backup.sh
```

Meaning:

Run backup.sh every day at 2:30 PM.

---

# Meaning of Special Characters

## *

Means

Every value

Example

```
* * * * *
```

Every minute.

---

## ,

Means

Multiple values.

Example

```
0 8,20 * * *
```

Runs at 8 AM and 8 PM.

---

## -

Means

Range.

Example

```
0 9-17 * * *
```

Runs every hour between 9 AM and 5 PM.

---

## */

Means

Step value.

Example

```
*/10 * * * *
```

Runs every 10 minutes.

---

# Common Examples

Every minute

```
* * * * *
```

Every five minutes

```
*/5 * * * *
```

Every hour

```
0 * * * *
```

Every day

```
0 0 * * *
```

Every Sunday

```
0 0 * * 0
```

Every weekday

```
0 9 * * 1-5
```

Every month

```
0 0 1 * *
```

Every year

```
0 0 1 1 *
```

---

# Where Cron is Used

System administrators use Cron for:

- Automatic backups
- Log cleanup
- Monitoring servers
- Database backups
- Email reports
- Restarting services
- Certificate renewal
- Security scans
- Disk cleanup
- Running shell scripts

---

# Cron Job Workflow

1. User creates a cron job.
2. Job is stored in the user's crontab.
3. Cron daemon checks schedules every minute.
4. When the scheduled time matches, Cron executes the command.
5. The process runs in the background.
6. Cron waits for the next scheduled execution.

---

# Cron Environment

Cron runs with a minimal environment.

This means:

- Environment variables may not be available.
- Relative paths may fail.
- Commands should use absolute paths.
- Scripts should have executable permissions.
- Redirect output to log files for debugging.

Example:

```bash
0 9 * * * /home/ubuntu/scripts/backup.sh >> /home/ubuntu/logs/backup.log 2>&1
```

Here:

- `>>` appends standard output to the log file.
- `2>&1` redirects standard error to the same log file.

---

# Cron vs Manual Execution

| Manual | Cron |
|--------|------|
| Started by user | Started automatically |
| Interactive | Background process |
| Needs user action | Fully automated |
| Good for one-time tasks | Best for recurring tasks |

---

# Cron Best Practices

- Always use absolute file paths.
- Test scripts manually before scheduling.
- Keep cron jobs simple.
- Log the output for troubleshooting.
- Avoid overlapping jobs unless intended.
- Review cron logs when jobs fail.
- Use meaningful comments in complex scripts.
- Verify the cron service is running.

---

# Advantages of Cron

- Saves time
- Fully automated
- Reliable
- Lightweight
- Easy to configure
- Runs in the background
- Reduces manual effort
- Ideal for production servers

---

# Limitations

- Executes with limited environment variables.
- Minute is the smallest scheduling interval.
- Failed jobs are not retried automatically.
- Long-running jobs may overlap if not managed.

---

# Real-World DevOps Examples

Daily database backup:

```bash
0 1 * * * /opt/scripts/db_backup.sh
```

Delete logs older than 30 days:

```bash
0 3 * * * find /var/log -type f -mtime +30 -delete
```

Check disk usage every hour:

```bash
0 * * * * /opt/scripts/disk_monitor.sh
```

Renew SSL certificates:

```bash
0 2 * * * certbot renew
```

Restart a service every Sunday:

```bash
0 4 * * 0 systemctl restart nginx
```

---

# Key Takeaways

- Cron is Linux's built-in task scheduler.
- Crontab stores scheduled jobs.
- The cron daemon checks jobs every minute.
- Five time fields define when a job runs.
- Proper paths, permissions, and logging are essential for reliable automation.
- Cron is a foundational tool used extensively in DevOps, cloud operations, and system administration.
