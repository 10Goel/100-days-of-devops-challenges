# Day 11 — Deploy a Website on Tomcat

## 📌 Challenge Overview

Day 11 of the 100 Days of DevOps challenge focused on installing and configuring an Apache Tomcat server and deploying a Java web application on it.

The task required deploying the provided `ROOT.war` application on **App Server 2 (`stapp02`)** and making the application accessible through the specified HTTP port.

---

## 🎯 Objectives

The following requirements had to be completed:

- Install Java/Tomcat on App Server 2.
- Configure Tomcat to listen on HTTP port `6400`.
- Deploy the provided `ROOT.war` file.
- Make the application available through the root URL.
- Start and verify the Tomcat service.
- Verify the deployed application using `curl`.

---

## 🖥️ Server Details

| Component | Details |
|---|---|
| Server | App Server 2 |
| Hostname | `stapp02` |
| Application Server | Apache Tomcat |
| HTTP Port | `6400` |
| WAR File | `ROOT.war` |
| Deployment Directory | `/var/lib/tomcat/webapps/` |
| Application URL | `http://stapp02:6400` |

---

## 🛠️ Implementation

### 1. Connect to App Server 2

```bash
ssh steve@stapp02
