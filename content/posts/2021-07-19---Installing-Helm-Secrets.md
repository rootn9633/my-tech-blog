---
title: Installing Helm Secrets
date: "2021-07-19T00:00:00.000Z"
template: "post"
draft: false
slug: "/posts/installing-helm-secrets"
category: "Homelab"
tags:
  - "Helm"
  - "Helm Secrets"
  - "Kubernets"
description: "How to install the helm-secrets plugin"
---

Some things are secret, and storing them as plain text in your helm chart is a bad idea. This is where [helm secrets](https://github.com/jkroepke/helm-secrets) comes in handy. It is a helm plugin that uses the [Mozilla Sops](https://github.com/mozilla/sops) tool to manages secrets.

## Install

Presuming helm is installed already, installing is simple with helm plugin.

```bash
helm plugin install https://github.com/jkroepke/helm-secrets --version v4.4.2
```

## Generate

Helm secrets supports many encryption methods, here I am using **PGP.**

First I generated a key pair:

```bash
gpg --gen-key
```

The generated keys could be checked by using

```bash
gpg --list-keys
```

Create a `.sops.yaml` file and insert the public key fingerprint:

```bash
creation_rules:
	- pgp: '<your-key-fingerprint>'
```

now you can encrypt by using:

```bash
helm secrets enc <filename>
```

Other useful commands are:

edit:

```bash
helm secrets edit <filename>
```

view:

```bash
helm secrets veiw <filename>
```

decrypt:

```bash
helm secrets dec <filename>
```

## Troubleshooting

### GPG timeout

When using `gpg --gen-key`:

```bash
gpg: agent_genkey failed: Timeout
Key generation failed: Timeout
```

It may be due to the system not having enough entropy. You could try moving the cursor, typing on the keyboard or making lots of hard drive IO

### helm secrets edit error

When using `helm secrets edit`:

```bash
Recovery failed because no master key was able to decrypt the file. In
order for SOPS to recover the file, at least one key has to be successful,
but none were.
Error: plugin "secrets" exited with error
```

Try resetting the terminal and make the gpg-agent ask for the password again:

```bash
reset
GPG_TTY=$(tty)
export GPG_TTY
```

ref: [https://github.com/zendesk/helm-secrets/issues/21](https://github.com/zendesk/helm-secrets/issues/21)

## Reference

- [https://developer.epages.com/blog/tech-stories/kubernetes-deployments-with-helm-secrets/](https://developer.epages.com/blog/tech-stories/kubernetes-deployments-with-helm-secrets/)
- [https://gist.github.com/twolfson/01d515258eef8bdbda4f](https://gist.github.com/twolfson/01d515258eef8bdbda4f)
