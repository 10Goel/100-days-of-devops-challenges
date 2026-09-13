# Day 45 --- Commands Reference

## 1. Connect to App Server 1

``` bash
ssh tony@stapp01
```

## 2. Navigate to the Dockerfile directory

``` bash
cd /opt/docker
```

## 3. Inspect the directory

``` bash
ls -la
```

## 4. Inspect the original Dockerfile

``` bash
cat Dockerfile
```

## 5. Edit the Dockerfile

``` bash
vi Dockerfile
```

## 6. Correct the Dockerfile

The important corrections were:

``` dockerfile
FROM httpd:2.4.43
```

instead of:

``` dockerfile
IMAGE httpd:2.4.43
```

And change each command-form `ADD sed ...` instruction to `RUN sed ...`.

The corrected command pattern is:

``` dockerfile
RUN sed -i "..." /usr/local/apache2/conf/httpd.conf
```

## 7. Verify the corrected file

``` bash
cat Dockerfile
```

## 8. Build the Docker image

``` bash
docker build -t day45 .
```

## 9. Verify the image

``` bash
docker images
```

Optional filtered verification:

``` bash
docker images | grep day45
```

------------------------------------------------------------------------

# Important Commands Explained

### `docker build`

``` bash
docker build -t day45 .
```

-   `docker build` --- builds an image from a Dockerfile.
-   `-t day45` --- assigns the image the tag/name `day45`.
-   `.` --- uses the current directory as the build context.

### `FROM`

``` dockerfile
FROM httpd:2.4.43
```

Defines the base image from which the new image is created.

### `RUN`

``` dockerfile
RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
```

Executes a command during the image build process.

### `COPY`

``` dockerfile
COPY html/index.html /usr/local/apache2/htdocs/
```

Copies a file from the build context into the image.

------------------------------------------------------------------------

# Troubleshooting Summary

  Problem                  Incorrect              Correct
  ------------------------ ---------------------- ---------------------
  Base image declaration   `IMAGE httpd:2.4.43`   `FROM httpd:2.4.43`
  Execute `sed` command    `ADD sed -i ...`       `RUN sed -i ...`
  Certificate copy         Valid `COPY`           Preserved
  Website content          Valid `COPY`           Preserved

------------------------------------------------------------------------

# Final Verification

``` bash
docker build -t day45 .
docker images
```

The build completed successfully and the `day45` image was created.
