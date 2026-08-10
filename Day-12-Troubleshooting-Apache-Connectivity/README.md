DevOps Challenge --- Troubleshoot Apache Connectivity on Port 3000

📌 Overview

This challenge involves troubleshooting an Apache web service on anapplication server in the Stratos Datacenter.

The monitoring system reports that the Apache service on stapp03is not reachable on TCP port 3000. The objective is to identifythe cause, restore connectivity, and verify that the service can beaccessed from the jump host.

Important: Do not modify the existing index.html file. Changingits contents can cause the challenge validation to fail.

🎯 Objective

Restore Apache connectivity on:

Server: stapp03
Service: Apache HTTP Server
Port: 3000/TCP
Test URL: http://stapp03:3000

The final validation should succeed from the jump host using:

curl http://stapp03:3000

🏗️ Environment
