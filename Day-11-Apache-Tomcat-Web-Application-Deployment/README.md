# Day 11 – Apache Tomcat Web Application Deployment

![DevOps](https://img.shields.io/badge/100%20Days%20of%20DevOps-Day%2011-blue)
![Apache Tomcat](https://img.shields.io/badge/Apache-Tomcat-orange)
![Linux](https://img.shields.io/badge/Linux-Server-red)

## 📌 Challenge Overview

The production support team of **xFusionCorp Industries** required a Java web application to be deployed on **App Server 2** in the **Stratos Datacenter**.

The task was to install and configure **Apache Tomcat**, deploy the provided `ROOT.war` application, configure the required HTTP port, start the Tomcat service, troubleshoot the startup issue, and verify that the application was accessible successfully.

The application was required to run on:

```text
App Server 2
stapp02
```

using HTTP port:

```text
6400
```

---

## 🎯 Objectives

1. Install Apache Tomcat on App Server 2.
2. Verify that Java is available.
3. Configure Tomcat to use HTTP port `6400`.
4. Deploy the provided `ROOT.war` application.
5. Ensure the application is deployed at the root context.
6. Start and verify the Tomcat service.
7. Troubleshoot the Tomcat port conflict.
8. Verify the deployed application using `curl`.

---

## 🖥️ Server Details

| Component | Details |
|---|---|
| Datacenter | Stratos Datacenter |
| Server | App Server 2 |
| Hostname | `stapp02` |
| Application Server | Apache Tomcat |
| HTTP Port | `6400` |
| Application Archive | `ROOT.war` |
| Tomcat Configuration | `/etc/tomcat/server.xml` |
| Tomcat Webapps Directory | `/var/lib/tomcat/webapps/` |
| WAR Deployment Path | `/var/lib/tomcat/webapps/ROOT.war` |
| Verification URL | `http://stapp02:6400` |

---

# 🔧 Implementation

## 1. Connect to App Server 2

Connect from the Jump Host:

```bash
ssh <app-server-user>@stapp02
```

Verify the hostname:

```bash
hostname
```

Expected:

```text
stapp02
```

---

## 2. Verify Java

Tomcat is a Java-based application server, so Java must be available before Tomcat can run.

```bash
java -version
```

---

## 3. Install Apache Tomcat

Install Tomcat using the package manager:

```bash
sudo dnf install tomcat -y
```

Verify the installation:

```bash
rpm -qa | grep tomcat
```

---

## 4. Configure Tomcat

The main Tomcat configuration file used in this task is:

```text
/etc/tomcat/server.xml
```

The HTTP Connector was configured to use the required port `6400`:

```xml
<Connector port="6400" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Verify the configured port:

```bash
sudo grep -n '6400' /etc/tomcat/server.xml
```

---

# ⚠️ Troubleshooting the Port Conflict

During the task, Tomcat initially failed to remain active. The logs showed:

```text
java.net.BindException: Address already in use
```

and:

```text
Failed to create server shutdown socket
```

The logs also showed:

```text
Initializing ProtocolHandler ["http-nio-6400"]
```

This indicated that the HTTP Connector had already bound to port `6400`, while the shutdown socket was attempting to use the same port.

The correct configuration is:

```text
HTTP Connector
      |
      +---- Port 6400
      |
      +---- Handles HTTP requests

Tomcat Shutdown Port
      |
      +---- Separate unused port
      |
      +---- Used internally by Tomcat
```

The required HTTP port was **not changed**. The shutdown port was configured separately to eliminate the conflict.

---

# 📦 Deploy the WAR File

The provided application archive was:

```text
ROOT.war
```

Copy it into Tomcat's web application directory:

```bash
sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/
```

Verify the deployment file:

```bash
ls -lh /var/lib/tomcat/webapps/
```

Expected:

```text
ROOT.war
```

---

# 🌐 Why `ROOT.war` Is Important

Tomcat normally uses the WAR filename to determine the application's context path.

For example:

```text
myapp.war
```

would normally be available at:

```text
http://stapp02:6400/myapp
```

However, `ROOT.war` is deployed as the root web application:

```text
/
```

Therefore, the application is accessed directly at:

```text
http://stapp02:6400
```

rather than:

```text
http://stapp02:6400/ROOT
```

---

# ▶️ Start Tomcat

Start the service:

```bash
sudo systemctl start tomcat
```

Check its status:

```bash
sudo systemctl status tomcat
```

A successful service should show:

```text
Active: active (running)
```

After configuration changes, restart Tomcat:

```bash
sudo systemctl restart tomcat
```

---

# 📝 Check Tomcat Logs

View recent Tomcat logs:

```bash
sudo journalctl -u tomcat -n 50 --no-pager
```

Important entries included:

```text
Initializing ProtocolHandler ["http-nio-6400"]
```

and:

```text
Deploying web application archive [/var/lib/tomcat/webapps/ROOT.war]
```

The startup failure was identified through:

```text
java.net.BindException: Address already in use
```

---

# 🔍 Verify Port 6400

Check whether a process is listening on the required port:

```bash
sudo ss -lntp | grep 6400
```

---

# 🧪 Verify the Application

Test locally:

```bash
curl http://localhost:6400
```

Test using the server hostname:

```bash
curl http://stapp02:6400
```

The command successfully returned the application's HTML content, confirming that Tomcat, the HTTP Connector, and the deployed application were working correctly.

---

# 📂 Final File Structure

```text
/etc/tomcat/
└── server.xml

/var/lib/tomcat/webapps/
└── ROOT.war
```

---

# 🏗️ Deployment Architecture

```text
                    Stratos Datacenter
                           |
                           v
                    App Server 2
                       stapp02
                           |
                           v
                    Apache Tomcat
                           |
             ┌─────────────┴─────────────┐
             |                           |
       HTTP Connector              Shutdown Port
          6400                    Separate Port
             |
             v
          ROOT.war
             |
             v
       Root Web Application
             |
             v
   http://stapp02:6400
             |
             v
        HTML Response
```

---

# 📚 Concepts Practiced

- Apache Tomcat
- Java runtime
- Java web applications
- WAR files
- `ROOT.war`
- Tomcat web application deployment
- Tomcat `server.xml`
- HTTP Connectors
- HTTP ports
- Tomcat shutdown ports
- Linux services
- `systemctl`
- `journalctl`
- Network port troubleshooting
- `ss`
- HTTP testing with `curl`
- Application deployment verification

---

# 🧠 Key Learning

The most important troubleshooting lesson was understanding the difference between the **Tomcat HTTP port** and the **Tomcat shutdown port**.

The HTTP port was required to be:

```text
6400
```

Therefore, it had to remain unchanged. The shutdown port needed to use a separate available port.

Using the same port for both caused:

```text
Address already in use
```

Once the ports were separated, Tomcat started successfully.

---

# 🛠️ Troubleshooting Workflow

```text
Tomcat service fails
        |
        v
Check service status
        |
        v
Read Tomcat logs
        |
        v
Identify BindException
        |
        v
Inspect port configuration
        |
        v
Separate HTTP and shutdown ports
        |
        v
Restart Tomcat
        |
        v
Verify service status
        |
        v
Verify port 6400
        |
        v
Test application using curl
```

A professional DevOps troubleshooting approach is to identify the root cause from the service status and logs before repeatedly restarting the service.

---

# ✅ Final Verification Checklist

| Requirement | Status |
|---|---|
| Connected to App Server 2 | ✅ |
| Java verified | ✅ |
| Apache Tomcat installed | ✅ |
| Tomcat configuration located | ✅ |
| HTTP port configured to `6400` | ✅ |
| `ROOT.war` deployed | ✅ |
| Tomcat shutdown-port conflict resolved | ✅ |
| Tomcat service started successfully | ✅ |
| Port `6400` verified | ✅ |
| Application tested with `curl` | ✅ |
| HTML response received | ✅ |
| Day 11 challenge completed | ✅ |

---

# 🏆 Final Result

The `ROOT.war` Java web application was successfully deployed on **App Server 2 (`stapp02`)** using Apache Tomcat.

Tomcat was configured to serve HTTP traffic on the required port:

```text
6400
```

The initial shutdown-port conflict was successfully identified and resolved without changing the required HTTP port.

The final verification command:

```bash
curl http://stapp02:6400
```

successfully returned the application's HTML content.

```text
Day 11 – Apache Tomcat Deployment

Java Verified                  ✅
Tomcat Installed               ✅
HTTP Port 6400 Configured      ✅
ROOT.war Deployed              ✅
Port Conflict Resolved         ✅
Tomcat Running                 ✅
Application Verified           ✅

Challenge Completed            ✅
```

---
