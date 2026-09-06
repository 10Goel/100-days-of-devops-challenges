# Day 40 - Commands

## 1. Connect to App Server 3

```bash
ssh <user>@stapp03
```

Example:

```bash
ssh banner@stapp03
```

## 2. Verify the Target Container

```bash
docker ps
```

```bash
docker ps --filter "name=kkloud"
```

## 3. Access the Running Container

```bash
docker exec -it kkloud bash
```

## 4. Update Package Metadata

```bash
apt update
```

## 5. Install Apache2

```bash
apt install -y apache2
```

## 6. Check Apache Listening Configuration

```bash
cat /etc/apache2/ports.conf
```

## 7. Change Apache Port from 80 to 5002

```bash
sed -i 's/^Listen 80$/Listen 5002/' /etc/apache2/ports.conf
```

Verify:

```bash
cat /etc/apache2/ports.conf
```

Expected:

```text
Listen 5002
```

## 8. Update the Default Virtual Host

```bash
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:5002>/' /etc/apache2/sites-available/000-default.conf
```

Verify:

```bash
grep -n "VirtualHost" /etc/apache2/sites-available/000-default.conf
```

Expected:

```text
<VirtualHost *:5002>
```

## 9. Validate Apache Configuration

```bash
apache2ctl configtest
```

Expected:

```text
Syntax OK
```

## 10. Start Apache

```bash
service apache2 start
```

Check status:

```bash
service apache2 status
```

## 11. Verify Port 5002

```bash
ss -lntp | grep 5002
```

## 12. Test Apache

```bash
curl -I http://localhost:5002
```

## 13. Exit the Container

```bash
exit
```

## 14. Verify the Container Is Still Running

```bash
docker ps --filter "name=kkloud"
```

# Complete Command Sequence

```bash
ssh banner@stapp03
docker ps
docker exec -it kkloud bash
apt update
apt install -y apache2
sed -i 's/^Listen 80$/Listen 5002/' /etc/apache2/ports.conf
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:5002>/' /etc/apache2/sites-available/000-default.conf
apache2ctl configtest
service apache2 start
ss -lntp | grep 5002
curl -I http://localhost:5002
exit
docker ps --filter "name=kkloud"
```

# Additional Useful Inspection Commands

### View Container Processes

```bash
docker top kkloud
```

### Inspect Container Configuration

```bash
docker inspect kkloud
```

### View Container Logs

```bash
docker logs kkloud
```

### Check Resource Usage

```bash
docker stats kkloud
```

### Execute a Single Command

```bash
docker exec kkloud ss -lntp
```
