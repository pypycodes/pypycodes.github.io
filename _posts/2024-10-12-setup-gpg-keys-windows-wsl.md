---
layout: post
title:  "How to Use a Windows-Created GPG Key in WSL"
date:   2024-10-12 10:00:00
categories: Architecture
tags: featured
# image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.JPG
---

If you've generated a GPG key on Windows and need to use it within your WSL (Windows Subsystem for Linux) environment, this guide will walk you through the steps to export the key from Windows, import it into WSL, and configure it for use in Git or other applications.

## Step 1: Export the GPG Key from Windows

First, we need to export the GPG key from the Windows environment.

1. Open a terminal on Windows (either PowerShell or Command Prompt).
2. List your GPG keys to find the key you want to export:

   ```bash
   gpg --list-secret-keys --keyid-format LONG
   ```

   Take note of the key ID for the key you want to export. It will appear after `sec   rsa4096/`.
3. Export the private key by running:

   ```bash
   gpg --export-secret-keys --armor KEY_ID > my-private-key.asc
   ```

   Replace `KEY_ID` with the actual GPG key ID. This will create a file named `my-private-key.asc` containing your private key in ASCII format.
4. If you also need the public key, export it as follows:

   ```bash
   gpg --export --armor KEY_ID > my-public-key.asc
   ```

## Step 2: Import the GPG Key into WSL

With the key exported, you can now import it into WSL.

1. Open your WSL terminal.
2. Navigate to the directory containing the `.asc` files. If the files are saved on the Windows side, they can be accessed through the `/mnt` directory. For example:

   ```bash
   cd /mnt/c/Users/YourUsername/PathToKeys/
   ```

3. Import the private key into WSL:

   ```bash
   gpg --import my-private-key.asc
   ```

4. If you exported the public key as well, import it with:

   ```bash
   gpg --import my-public-key.asc
   ```

5. After importing, set the key as trusted:

   ```bash
   gpg --edit-key KEY_ID
   ```

   Inside the GPG prompt, type:

   ```bash
   trust
   ```

   Choose the appropriate trust level (usually `5` for full trust) and then type `quit` to exit.

## Step 3: Verify the Key in WSL

To confirm the key was successfully imported, list the keys in WSL:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Your imported key should be displayed in the list.

## Step 4: Configure Git to Use the GPG Key (Optional)

If you want to use this GPG key to sign commits in Git, configure Git with the following steps:

1. Set the key as the signing key:

   ```bash
   git config --global user.signingkey KEY_ID
   ```

   Replace `KEY_ID` with your GPG key ID.

2. Enable automatic GPG signing for Git commits:

   ```bash
   git config --global commit.gpgSign true
   ```

## Conclusion

By following these steps, you can seamlessly use your GPG key created on Windows inside your WSL environment. This setup is particularly useful if you use Git on both Windows and WSL and want to keep your GPG key consistent across environments.
