# Day 11 – Apache Tomcat Notes

## 📌 Overview

Day 11 focused on deploying a Java web application using **Apache Tomcat** on **App Server 2 (`stapp02`)**.

The main concepts covered were:

- Java and Tomcat
- Apache Tomcat
- WAR files
- `ROOT.war`
- Tomcat configuration
- HTTP Connectors
- HTTP ports
- Tomcat shutdown ports
- Linux service management
- `systemctl`
- `journalctl`
- Port troubleshooting
- `curl`
- Application deployment

The required HTTP port for the application was:

```text
6400
```

---

# 1. What is Apache Tomcat?

Apache Tomcat is an open-source Java application server and servlet container.

It provides an environment in which Java web applications can run.

A simplified architecture is:

```text
Client
   |
   | HTTP Request
   v
Apache Tomcat
   |
   v
Java Web Application
   |
   v
HTTP Response
```

Tomcat commonly works with:

- Java Servlets
- JSP
- Java web applications
- WAR files

---

# 2. Why Does Tomcat Need Java?

Tomcat itself is implemented using Java.

Therefore, Java must be available on the server for Tomcat to execute.

The relationship is:

```text
Linux
  |
  v
Java Runtime
  |
  v
Apache Tomcat
  |
  v
Java Web Application
```

Java was verified using:

```bash
java -version
```

---

# 3. What is a WAR File?

WAR stands for:

```text
Web Application Archive
```

A WAR file is a packaged Java web application.

A WAR can contain:

```text
application.war
│
├── WEB-INF/
│   ├── web.xml
│   ├── classes/
│   └── lib/
│
├── HTML files
├── JSP files
├── CSS
├── JavaScript
└── Other application resources
```

Tomcat can deploy a WAR file placed inside its web application directory.

In this challenge, the application archive was:

```text
ROOT.war
```

---

# 4. What is `ROOT.war`?

Tomcat uses the WAR filename to determine the application's context path.

For example:

```text
myapp.war
```

would normally be available at:

```text
http://server:6400/myapp
```

But:

```text
ROOT.war
```

is special.

It is deployed as the root application:

```text
/
```

Therefore:

```text
ROOT.war
```

is accessible using:

```text
http://stapp02:6400
```

rather than:

```text
http://stapp02:6400/ROOT
```

This is why the challenge specifically used `ROOT.war`.

---

# 5. Tomcat Web Application Directory

The Tomcat web application directory used in this environment was:

```text
/var/lib/tomcat/webapps/
```

The application was deployed as:

```text
/var/lib/tomcat/webapps/ROOT.war
```

The deployment process is:

```text
ROOT.war
   |
   v
/var/lib/tomcat/webapps/
   |
   v
Tomcat detects WAR
   |
   v
Tomcat deploys application
   |
   v
Root context (/)
   |
   v
http://stapp02:6400
```

---

# 6. What is `server.xml`?

Tomcat's primary configuration file is:

```text
/etc/tomcat/server.xml
```

This file contains important server-level configuration such as:

- HTTP connectors
- HTTPS connectors
- AJP connectors
- Shutdown port
- Connection settings
- Engine configuration
- Host configuration

For this challenge, the HTTP Connector and shutdown port configuration were especially important.

---

# 7. What is a Tomcat Connector?

A Connector is responsible for accepting connections from clients.

For HTTP, a connector can look like:

```xml
<Connector port="6400" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

The important part is:

```text
port="6400"
```

This means Tomcat listens for HTTP traffic on TCP port `6400`.

The request flow is:

```text
Client
   |
   | HTTP Request
   | TCP Port 6400
   v
Tomcat HTTP Connector
   |
   v
ROOT.war Application
   |
   v
HTTP Response
```

---

# 8. What is a Port?

A port is a logical communication endpoint used by network applications.

A server can run multiple services because different services can listen on different ports.

For example:

```text
22    → SSH
80    → HTTP
443   → HTTPS
3306  → MySQL/MariaDB
6400  → Tomcat HTTP application
```

A port cannot normally be bound by multiple processes using the same address/protocol combination at the same time.

This is extremely important when troubleshooting services.

---

# 9. HTTP Port vs Tomcat Shutdown Port

This was the most important troubleshooting concept in Day 11.

Tomcat has different types of ports for different purposes.

## HTTP Connector Port

The HTTP Connector receives normal HTTP requests from clients.

For this challenge:

```text
HTTP Port = 6400
```

Therefore:

```text
curl http://stapp02:6400
```

connects to the Tomcat HTTP Connector.

---

## Tomcat Shutdown Port

Tomcat also has a **shutdown port**.

The shutdown port is not the port through which normal web traffic is served.

It is a control socket used by Tomcat for shutdown communication.

Conceptually:

```text
HTTP Connector
      |
      +---- Port 6400
      |
      +---- Receives HTTP requests


Shutdown Socket
      |
      +---- Separate port
      |
      +---- Used for shutdown/control communication
```

These ports have completely different purposes.

---

# 10. The Shutdown Port Conflict – What Happened?

During the Day 11 task, Tomcat initially showed:

```text
Active: inactive
```

The service was not remaining active after startup.

Instead of immediately changing the required HTTP port, the Tomcat logs were checked.

The logs contained:

```text
SEVERE ... Failed to create server shutdown socket
```

and:

```text
java.net.BindException: Address already in use
```

This was the key error.

---

# 11. Understanding `Address already in use`

The error:

```text
java.net.BindException: Address already in use
```

means that Tomcat tried to bind a socket to an address/port that was already occupied.

In this particular situation, the sequence was:

```text
Tomcat starts
     |
     v
Tomcat initializes HTTP Connector
     |
     v
HTTP Connector successfully binds to port 6400
     |
     v
Tomcat tries to create shutdown socket
     |
     v
Shutdown socket also attempts to use port 6400
     |
     v
Port 6400 is already occupied
     |
     v
BindException
     |
     v
Tomcat shuts itself down
```

This explains why the logs could show that the HTTP server had successfully started and then immediately show a shutdown-related failure.

---

# 12. Why Did Tomcat Become Inactive?

This is an important point.

The logs showed:

```text
Server startup in [1410] milliseconds
```

which might initially make it look as if Tomcat started successfully.

However, immediately after that, the logs showed:

```text
Failed to create server shutdown socket
```

followed by:

```text
java.net.BindException: Address already in use
```

Tomcat then stopped:

```text
Stopping service [Catalina]
```

and systemd reported:

```text
tomcat.service: Deactivated successfully.
```

Therefore, the service eventually became:

```text
inactive
```

The problem was not that Tomcat could not initialize the HTTP Connector.

The problem was that Tomcat could not create its shutdown socket because the required port was already occupied.

---

# 13. Why We Could Not Simply Change Port 6400

The challenge specifically required:

```text
HTTP Port = 6400
```

Therefore, changing the HTTP Connector to another port would violate the task requirement.

The correct solution was:

```text
Keep HTTP Connector → 6400

Change/fix Shutdown Port → Separate unused port
```

The final architecture was:

```text
              Apache Tomcat
                    |
          ┌─────────┴─────────┐
          |                   |
          v                   v
   HTTP Connector       Shutdown Socket
      Port 6400          Separate Port
          |                   |
          v                   v
   Browser / curl       Tomcat control
```

This is the correct way to solve the conflict.

---

# 14. Why Must the Shutdown Port Be Different?

Imagine two applications trying to occupy the same parking space:

```text
Process A
   |
   +---- wants Port 6400
   |
   v
[ Port 6400 ]

Process B
   |
   +---- also wants Port 6400
   |
   v
[ Port 6400 ]
```

The operating system cannot allow both sockets to bind to the same address/port combination in the normal case.

Therefore:

```text
HTTP Connector → 6400
Shutdown Socket → Another available port
```

is required.

---

# 15. Important Difference Between HTTP and Shutdown Ports

| Feature | HTTP Connector | Shutdown Port |
|---|---|---|
| Purpose | Receives HTTP requests | Tomcat shutdown/control |
| Used by browser/curl | Yes | No |
| Required application port | `6400` | Separate |
| Handles web traffic | Yes | No |
| Should conflict with HTTP port | No | No |

This distinction is extremely important for Tomcat troubleshooting.

---

# 16. How the Logs Helped Us

Instead of guessing, the logs provided the exact root cause.

The important commands were:

```bash
sudo systemctl status tomcat
```

and:

```bash
sudo journalctl -u tomcat
```

The important error was:

```text
java.net.BindException: Address already in use
```

The logs also showed:

```text
Initializing ProtocolHandler ["http-nio-6400"]
```

This told us that the HTTP Connector was using port `6400`.

Then:

```text
Failed to create server shutdown socket
```

identified the shutdown socket as the component causing the problem.

This is a very important DevOps troubleshooting principle:

> Read the logs and identify the failing component before changing configuration.

---

# 17. What is `systemctl`?

`systemctl` is the command used to manage services controlled by `systemd`.

For example:

### Start Tomcat

```bash
sudo systemctl start tomcat
```

### Stop Tomcat

```bash
sudo systemctl stop tomcat
```

### Restart Tomcat

```bash
sudo systemctl restart tomcat
```

### Check status

```bash
sudo systemctl status tomcat
```

### Enable at boot

```bash
sudo systemctl enable tomcat
```

---

# 18. What Does `active (running)` Mean?

When:

```bash
sudo systemctl status tomcat
```

shows:

```text
Active: active (running)
```

Tomcat is currently running as a systemd service.

If it shows:

```text
inactive (dead)
```

Tomcat is not running.

If it shows:

```text
failed
```

the service attempted to start but encountered a failure.

In Day 11, Tomcat initially became inactive because of the shutdown-port conflict.

---

# 19. What is `journalctl`?

`journalctl` is used to inspect logs collected by `systemd`.

For Tomcat:

```bash
sudo journalctl -u tomcat
```

This displays logs related to the Tomcat service.

For the latest 50 lines:

```bash
sudo journalctl -u tomcat -n 50 --no-pager
```

Logs are extremely useful for diagnosing:

- Port conflicts
- Permission problems
- Configuration errors
- Java errors
- Application deployment failures
- Service startup failures

---

# 20. What is `curl`?

`curl` is a command-line tool used to communicate with web servers.

For this challenge:

```bash
curl http://stapp02:6400
```

This sends an HTTP request to:

```text
Hostname → stapp02
Port     → 6400
Protocol → HTTP
```

The application returned HTML content, confirming that the web application was working.

---

# 21. Why `curl` Is a Good Verification Tool

Using `curl` allows us to verify the application without opening a browser.

A successful response confirms several things:

```text
Hostname resolves
       |
       v
Network connection works
       |
       v
Port 6400 is reachable
       |
       v
Tomcat is listening
       |
       v
ROOT.war is deployed
       |
       v
Application responds
       |
       v
HTML content returned
```

Therefore, `curl` is an excellent DevOps verification tool.

---

# 22. `localhost` vs `stapp02`

Two useful tests are:

```bash
curl http://localhost:6400
```

and:

```bash
curl http://stapp02:6400
```

### `localhost`

`localhost` refers to the current machine.

```text
localhost → this server
```

This test helps determine whether the application is running locally.

### `stapp02`

`stapp02` refers to the App Server 2 hostname.

```bash
curl http://stapp02:6400
```

This provides a test using the server's hostname.

---

# 23. What Happens When Tomcat Deploys `ROOT.war`?

When Tomcat starts, it checks its web application directory:

```text
/var/lib/tomcat/webapps/
```

If it finds:

```text
ROOT.war
```

it deploys the application.

The logs can show:

```text
Deploying web application archive [/var/lib/tomcat/webapps/ROOT.war]
```

and then:

```text
Deployment of web application archive [/var/lib/tomcat/webapps/ROOT.war] has finished
```

This indicates that Tomcat successfully deployed the application.

---

# 24. Complete Tomcat Startup Flow

The complete startup process can be visualized as:

```text
systemctl start tomcat
          |
          v
       Java starts
          |
          v
    Tomcat initializes
          |
          v
   Reads server.xml
          |
          v
Creates HTTP Connector
          |
          v
   Binds port 6400
          |
          v
Deploys ROOT.war
          |
          v
Creates shutdown socket
          |
          v
All ports available?
      /         \
    Yes          No
     |            |
     v            v
Tomcat runs    BindException
                  |
                  v
             Tomcat stops
```

This flow explains exactly what happened during the troubleshooting process.

---

# 25. Tomcat Configuration Paths

Important paths from this challenge:

## Main configuration

```text
/etc/tomcat/server.xml
```

## Web application directory

```text
/var/lib/tomcat/webapps/
```

## Deployed application

```text
/var/lib/tomcat/webapps/ROOT.war
```

## Original WAR location

```text
/tmp/ROOT.war
```

---

# 26. Important Commands Learned

## Check Java

```bash
java -version
```

## Install Tomcat

```bash
sudo dnf install tomcat -y
```

## Check Tomcat package

```bash
rpm -qa | grep tomcat
```

## Start Tomcat

```bash
sudo systemctl start tomcat
```

## Restart Tomcat

```bash
sudo systemctl restart tomcat
```

## Check Tomcat status

```bash
sudo systemctl status tomcat
```

## View Tomcat logs

```bash
sudo journalctl -u tomcat
```

## View recent logs

```bash
sudo journalctl -u tomcat -n 50 --no-pager
```

## Check port 6400

```bash
sudo ss -lntp | grep 6400
```

## Check Tomcat HTTP configuration

```bash
sudo grep -n '6400' /etc/tomcat/server.xml
```

## Check deployed WAR

```bash
ls -lh /var/lib/tomcat/webapps/
```

## Test application

```bash
curl http://localhost:6400
```

## Test using hostname

```bash
curl http://stapp02:6400
```

---

# 27. Real-World DevOps Troubleshooting Pattern

Day 11 demonstrates a very important troubleshooting methodology.

When a service fails:

```text
1. Check service status
        ↓
2. Check logs
        ↓
3. Identify exact error
        ↓
4. Determine affected component
        ↓
5. Inspect configuration
        ↓
6. Fix the root cause
        ↓
7. Restart service
        ↓
8. Verify service
        ↓
9. Test application
```

For this challenge:

```text
Tomcat inactive
      ↓
systemctl status
      ↓
journalctl
      ↓
BindException
      ↓
Address already in use
      ↓
Shutdown socket conflict
      ↓
Keep HTTP port 6400
      ↓
Use separate shutdown port
      ↓
Restart Tomcat
      ↓
Tomcat active
      ↓
curl
      ↓
HTML response
```

---

# 28. Important Troubleshooting Lesson

A common mistake would be:

```text
Tomcat failed
      |
      v
Change HTTP port 6400
```

That would be incorrect because the challenge explicitly requires HTTP port `6400`.

The correct thought process is:

```text
Required HTTP port = 6400
           |
           v
Do not change it
           |
           v
Find what else is using 6400
           |
           v
Identify shutdown socket conflict
           |
           v
Separate shutdown port
```

This is an important example of fixing the **root cause** instead of changing a required application configuration.

---

# 29. Key Takeaways

## Tomcat

Tomcat is a Java-based application server used to run Java web applications.

## Java

Tomcat requires Java to execute.

## WAR

A WAR file packages a Java web application for deployment.

## ROOT.war

`ROOT.war` is deployed at the root context `/`.

## HTTP Connector

The HTTP Connector accepts HTTP requests.

For this task:

```text
HTTP → Port 6400
```

## Shutdown Port

The shutdown port is used for Tomcat control/shutdown communication.

It must not conflict with the HTTP Connector.

## `BindException`

```text
Address already in use
```

means a socket attempted to bind to a port that was already occupied.

## `systemctl`

Used to manage the Tomcat service.

## `journalctl`

Used to inspect service logs and troubleshoot failures.

## `curl`

Used to test the deployed web application from the command line.

---

# 30. Final Day 11 Architecture

```text
                         App Server 2
                            stapp02
                               |
                               v
                        Apache Tomcat
                               |
              ┌────────────────┴────────────────┐
              |                                 |
              v                                 v
      HTTP Connector                    Shutdown Socket
          Port 6400                     Separate Port
              |
              v
           ROOT.war
              |
              v
       Root Context (/)
              |
              v
     http://stapp02:6400
              |
              v
        HTML Response
```

---

# 31. Final Verification

The application was successfully verified using:

```bash
curl http://stapp02:6400
```

The command returned the expected HTML content.

Therefore:

```text
Java                         ✅
Tomcat                       ✅
server.xml configured        ✅
HTTP port 6400               ✅
ROOT.war deployed            ✅
Shutdown conflict resolved   ✅
Tomcat service running       ✅
Application responding       ✅
```

---

# 🏆 Day 11 Learning Summary

Day 11 was not only about installing Tomcat.

The challenge provided practical experience with:

```text
Java
  ↓
Tomcat
  ↓
server.xml
  ↓
HTTP Connector
  ↓
Port 6400
  ↓
WAR deployment
  ↓
ROOT.war
  ↓
Shutdown socket
  ↓
Port conflict troubleshooting
  ↓
systemctl
  ↓
journalctl
  ↓
curl verification
```

The most important lesson was:

> **When a service fails, do not immediately change configuration randomly. Read the logs, identify the exact component causing the failure, preserve the requirements of the task, fix the root cause, and then verify the complete application flow.**

---
