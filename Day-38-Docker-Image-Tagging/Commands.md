# Day 38 - Commands

## 1. Connect to App Server 2

```bash
ssh steve@stapp02
```

> Use the SSH user provided by the KodeKloud environment if it differs.

---

## 2. Pull the Required Docker Image

```bash
sudo docker pull busybox:musl
```

---

## 3. Verify the Pulled Image

```bash
sudo docker images busybox
```

---

## 4. Create the New Image Tag

```bash
sudo docker tag busybox:musl busybox:blog
```

---

## 5. Verify Both Tags

```bash
sudo docker images busybox
```

---

## Complete Command Sequence

```bash
ssh steve@stapp02

sudo docker pull busybox:musl

sudo docker tag busybox:musl busybox:blog

sudo docker images busybox
```
