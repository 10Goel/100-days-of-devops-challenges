# Day 45 --- Troubleshoot and Fix a Dockerfile

## 📌 Challenge Overview

This challenge focused on troubleshooting a broken Dockerfile and
successfully building a Docker image without changing the required base
image, application data, or valid configuration.

### Environment

-   **Server:** App Server 1 (`stapp01`)
-   **Directory:** `/opt/docker`
-   **Base Image:** `httpd:2.4.43`
-   **Image Name:** `day45`
-   **Application:** Apache HTTP Server
-   **Configured Port:** `8080`

------------------------------------------------------------------------

## 🎯 Objectives

The task required:

1.  Locate the Dockerfile under `/opt/docker`.
2.  Identify the syntax and instruction errors.
3.  Correct the Dockerfile.
4.  Preserve the specified base image and existing application data.
5.  Successfully build the Docker image.

------------------------------------------------------------------------

## 🔎 Original Dockerfile Problems

The Dockerfile contained several incorrect Dockerfile instructions.

### 1. Invalid `IMAGE` instruction

**Incorrect:**

``` dockerfile
IMAGE httpd:2.4.43
```

**Correct:**

``` dockerfile
FROM httpd:2.4.43
```

`FROM` specifies the base image for a Docker build. `IMAGE` is not a
valid Dockerfile instruction.

### 2. `ADD` incorrectly used to execute `sed`

The original file used statements such as:

``` dockerfile
ADD sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

This is incorrect because `ADD` is intended to copy files/directories
into an image. It does not execute shell commands.

**Correct:**

``` dockerfile
RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

The same correction was applied to all four `sed` commands.

### 3. Existing `COPY` instructions were preserved

The certificate and application files were already being copied
correctly, so their source data and destinations were not changed:

``` dockerfile
COPY certs/server.crt /usr/local/apache2/conf/server.crt
COPY certs/server.key /usr/local/apache2/conf/server.key
COPY html/index.html /usr/local/apache2/htdocs/
```

This follows the requirement not to change existing data such as
`index.html`.

------------------------------------------------------------------------

## ✅ Corrected Dockerfile

``` dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i 's/LoadModule\ ssl_module\ modules\/mod_ssl.so/#&/g' /usr/local/apache2/conf/httpd.conf

RUN sed -i 's/LoadModule\ socache_shmcb_module\ modules\/mod_socache_shmcb.so/#&/g' /usr/local/apache2/conf/httpd.conf

RUN sed -i 's/Include\ conf\/extra\/httpd-ssl.conf/#&/g' /usr/local/apache2/conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/
```

------------------------------------------------------------------------

## 🛠️ Solution Workflow

``` text
App Server 1
    │
    ├── /opt/docker
    │      └── Dockerfile
    │
    ├── Inspect Dockerfile
    │
    ├── Identify invalid instructions
    │      ├── IMAGE → FROM
    │      └── ADD sed → RUN sed
    │
    ├── Preserve valid COPY instructions
    │
    └── docker build
             │
             ▼
        day45 image
```

------------------------------------------------------------------------

## 🧪 Build Verification

The corrected Dockerfile was successfully built using:

``` bash
docker build -t day45 .
```

Verify the resulting image:

``` bash
docker images
```

The completed image should appear with the repository/name:

``` text
day45
```

------------------------------------------------------------------------

## 🧠 Key DevOps Concepts

-   Dockerfile syntax
-   Docker base images
-   `FROM`
-   `RUN`
-   `COPY`
-   `ADD` vs `COPY`
-   Apache HTTP Server configuration
-   `sed` for configuration modification
-   Docker image building
-   Docker build troubleshooting
-   Preserving application data during image troubleshooting

------------------------------------------------------------------------

## 💡 Key Learning

A Dockerfile instruction must match the operation being performed:

-   **`FROM`** → choose a base image
-   **`RUN`** → execute commands while building the image
-   **`COPY`** → copy files/directories into the image
-   **`ADD`** → copy files with additional archive/URL behavior, but it
    should not be used as a substitute for `RUN`

This challenge reinforced the importance of reading Docker build errors
carefully instead of changing working configuration unnecessarily.

------------------------------------------------------------------------

## ✅ Completion Status

**Day 45 --- Completed Successfully 🎉**

The Dockerfile was corrected without changing the required base image or
application data, and the Docker image was built successfully.
