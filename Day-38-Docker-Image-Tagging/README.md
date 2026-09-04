# Day 38 - Docker Image Tagging

## 🎯 Task Overview

The Nautilus DevOps team is preparing an application environment for testing. The task was to pull a specific Docker image on **App Server 2** and create an additional tag for the same image.

### Requirements

- Pull the `busybox:musl` image.
- Perform the task on **App Server 2**.
- Create a new Docker image tag named `busybox:blog`.
- Verify that both tags point to the same image.

---

## 🛠️ Technologies Used

- Docker
- Linux
- SSH

---

## 📋 Implementation

The required Docker image was first pulled from the container registry:

```bash
sudo docker pull busybox:musl
```

A new tag was then created from the existing image:

```bash
sudo docker tag busybox:musl busybox:blog
```

Finally, the available BusyBox images were verified:

```bash
sudo docker images busybox
```

Both `busybox:musl` and `busybox:blog` should reference the same Docker image ID.

---

## 🔍 Verification

Example output:

```text
REPOSITORY   TAG    IMAGE ID
busybox      musl   <IMAGE_ID>
busybox      blog   <IMAGE_ID>
```

The matching image IDs confirm that Docker created an additional tag rather than duplicating the image.

---

## 📚 Key Learning

- Docker image tags are human-readable references to image versions.
- Multiple tags can point to the same Docker image ID.
- `docker tag` creates an additional reference to an existing local image.
- Re-tagging an image does not create another copy of its layers.
- Docker image tagging is commonly used for versioning and preparing images for deployment to different environments or registries.

---

## ✅ Task Status

**DevOps Day 38 Challenge: Successfully Completed** 🎉
