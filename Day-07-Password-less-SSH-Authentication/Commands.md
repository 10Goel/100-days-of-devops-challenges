# Commands Used - Day 07

## Check Existing SSH Keys

```bash
ls -la ~/.ssh
```

---

## Generate SSH Key Pair

```bash
ssh-keygen -t rsa -b 4096
```

Options:

- RSA Algorithm
- 4096-bit key size

Press Enter for:

- Default file location
- Empty passphrase

---

## Copy Public Key to App Server 1

```bash
ssh-copy-id tony@stapp01
```

---

## Copy Public Key to App Server 2

```bash
ssh-copy-id steve@stapp02
```

---

## Copy Public Key to App Server 3

```bash
ssh-copy-id banner@stapp03
```

---

## Verify Password-less Login

```bash
ssh tony@stapp01
exit

ssh steve@stapp02
exit

ssh banner@stapp03
exit
```

---

## Display Command History

```bash
history
```

---

## View SSH Keys

```bash
ls ~/.ssh
```

---

## Display Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

---

## Display Private Key (Read Only)

```bash
cat ~/.ssh/id_rsa
```

> Never share the private key with anyone.
