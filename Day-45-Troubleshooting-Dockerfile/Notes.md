# Day 45 --- Dockerfile Troubleshooting Notes

## 1. What is a Dockerfile?

A Dockerfile is a text file containing instructions used by Docker to
build an image.

Example:

``` dockerfile
FROM ubuntu:22.04
RUN apt-get update
COPY app.sh /app/
CMD ["bash", "/app/app.sh"]
```

Each instruction creates a layer or contributes configuration to the
resulting image.

------------------------------------------------------------------------

# 2. Important Dockerfile Instructions

## FROM

Syntax:

``` dockerfile
FROM <image>:<tag>
```

Purpose:

-   Defines the base image.
-   Normally appears at the beginning of a Dockerfile.
-   Every Docker build starts from a base image.

Day 45 example:

``` dockerfile
FROM httpd:2.4.43
```

### Important

`IMAGE` is **not** a valid Dockerfile instruction.

Incorrect:

``` dockerfile
IMAGE httpd:2.4.43
```

Correct:

``` dockerfile
FROM httpd:2.4.43
```

------------------------------------------------------------------------

# 3. RUN

Syntax:

``` dockerfile
RUN <command>
```

Purpose:

Executes a command while building the image.

Example:

``` dockerfile
RUN apt-get update
```

Day 45:

``` dockerfile
RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

The command modifies Apache's configuration during the image build.

------------------------------------------------------------------------

# 4. COPY

Syntax:

``` dockerfile
COPY <source> <destination>
```

Purpose:

Copies files or directories from the Docker build context into the
image.

Example:

``` dockerfile
COPY html/index.html /usr/local/apache2/htdocs/
```

For Day 45, the existing certificate and website-copy instructions were
preserved.

------------------------------------------------------------------------

# 5. ADD vs COPY

Both can copy files into an image, but they are not interchangeable in
every situation.

### COPY

Designed specifically for copying files/directories.

``` dockerfile
COPY app.conf /etc/app/
```

### ADD

Has additional behavior such as archive extraction.

``` dockerfile
ADD application.tar.gz /opt/app/
```

### Critical point from Day 45

Neither `ADD` nor `COPY` should be used to execute shell commands.

Incorrect:

``` dockerfile
ADD sed -i "..." /path/to/file
```

Correct:

``` dockerfile
RUN sed -i "..." /path/to/file
```

------------------------------------------------------------------------

# 6. Why `sed` Was Used

`sed` is a stream editor commonly used in Linux automation to perform
text substitutions.

General pattern:

``` bash
sed -i 's/old/new/g' file
```

Where:

-   `s` = substitute
-   `old` = text to search for
-   `new` = replacement
-   `g` = replace all matching occurrences
-   `-i` = modify the file in place

Day 45 example:

``` bash
sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

This changes Apache's listening port from `80` to `8080`.

------------------------------------------------------------------------

# 7. Apache Configuration Changes

The Dockerfile modified the Apache configuration to:

### Change the listening port

``` text
Listen 80
```

to:

``` text
Listen 8080
```

### Disable SSL module configuration

The `LoadModule ssl_module` line was commented out.

### Disable the `socache_shmcb` module

The corresponding `LoadModule socache_shmcb_module` line was commented
out.

### Disable the SSL configuration include

The SSL configuration include was commented out.

These changes were already specified by the challenge and therefore were
preserved rather than redesigned.

------------------------------------------------------------------------

# 8. Docker Build Context

When running:

``` bash
docker build -t day45 .
```

the final `.` means:

> Use the current directory as the Docker build context.

Therefore, files referenced by:

``` dockerfile
COPY certs/server.crt ...
COPY certs/server.key ...
COPY html/index.html ...
```

must be available within the build context.

------------------------------------------------------------------------

# 9. Docker Image Build Process

Conceptually:

``` text
Dockerfile
    │
    ▼
Docker Build Context
    │
    ▼
Parse Dockerfile
    │
    ▼
FROM httpd:2.4.43
    │
    ▼
Execute RUN instructions
    │
    ▼
COPY application files
    │
    ▼
Create image layers
    │
    ▼
day45 image
```

------------------------------------------------------------------------

# 10. Common Dockerfile Troubleshooting Checklist

When `docker build` fails:

### Step 1 --- Read the first error

``` bash
docker build -t day45 .
```

Do not immediately modify multiple lines.

### Step 2 --- Check Dockerfile syntax

``` bash
cat Dockerfile
```

Look for:

-   Invalid instructions
-   Incorrect paths
-   Missing files
-   Invalid quoting
-   Incorrect command syntax

### Step 3 --- Check build-context files

``` bash
ls -la
find . -maxdepth 2 -type f
```

### Step 4 --- Verify base image

Make sure the required image and tag are unchanged.

### Step 5 --- Rebuild

``` bash
docker build -t day45 .
```

### Step 6 --- Verify

``` bash
docker images
```

------------------------------------------------------------------------

# 11. Day 45 Error → Fix Summary

  -----------------------------------------------------------------------
  Error                   Why it was wrong        Fix
  ----------------------- ----------------------- -----------------------
  `IMAGE httpd:2.4.43`    `IMAGE` is not a        Use `FROM`
                          Dockerfile instruction  

  `ADD sed -i ...`        `ADD` copies data; it   Use `RUN sed -i ...`
                          does not execute        
                          commands                

  Certificate `COPY`      Already valid           Keep unchanged

  `index.html` `COPY`     Existing application    Keep unchanged
                          data must not be        
                          changed                 
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 12. Key DevOps Takeaways

### Principle 1 --- Read the Dockerfile before changing it

Understand what each instruction is supposed to do.

### Principle 2 --- Use the correct instruction for the job

``` text
FROM  → Base image
RUN   → Execute command
COPY  → Copy files
ADD   → Copy with additional Docker-specific behavior
CMD   → Default runtime command
ENTRYPOINT → Main executable
ENV   → Environment variable
EXPOSE → Document intended container port
WORKDIR → Working directory
```

### Principle 3 --- Fix only what is broken

The challenge explicitly required preserving:

-   Base image
-   Valid configuration
-   Existing data

Therefore, the solution should be a targeted correction rather than a
complete Dockerfile rewrite.

------------------------------------------------------------------------

# 13. Final Takeaway

Day 45 demonstrates a very practical DevOps skill:

> **Troubleshoot the build error, understand the purpose of each
> Dockerfile instruction, and make the smallest valid change required to
> restore a successful build.**

The most important distinction from this challenge is:

``` dockerfile
RUN
```

**executes commands**, while:

``` dockerfile
COPY
ADD
```

**bring files/data into the image**.

Understanding this distinction is fundamental to writing and
troubleshooting Dockerfiles.
