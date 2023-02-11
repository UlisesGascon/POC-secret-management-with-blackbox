# POC Secret management with Blackbox

**Store secrets securely in Git using strong encryption without compromising the developer experience.**


### :crystal_ball: About

[Blackbox](https://github.com/StackExchange/blackbox) is a great tool for securely storing secrets within Git. It helps streamline the process of working with secrets while ensuring the highest level of security possible. With its easy-to-use PGP encryption and automated processes, Blackbox ensures developers have a safe and efficient way of managing their secrets.


### :tv: Tutorial 

[![promotional banner from youtube](.github/img/banner.png)](https://www.youtube.com/watch?v=beJT7wMvJo4)

Blog post available in [Dev.to](https://dev.to/ulisesgascon/safely-store-secrets-in-git-using-blackbox-419c) and [personal blog](https://blog.ulisesgascon.com/safely-store-secrets-in-git-using-blackbox)


## ⚙️ Instalation
In mac the best way is to use [Brew](https://brew.sh/), so you should have Brew installed in your machine, in the first place.

Then, with this command, install Blackbox:

```
brew install blackbox
````

In case that you are using a different platform please check the [official documentation](https://github.com/StackExchange/blackbox#installation-instructions).

## 🔐 Usage / How to

### Add users
♦️ Note: you need to have the user's pgp public key in your pgp keychain and you MUST be an ADMIN to run this command (you can't add yourself)

```
# Add the new key to the project
blackbox_addadmin {email}
# Re-encrypt the files including the new user
blackbox_update_all_files
```

### Remove users

```bash
# Remove the user from the project
blackbox_removeadmin {email}
# Re-encrypt the files without the previous key
blackbox_update_all_files
```

### Add a new file

By default, Backbox won't encrypt the new files, so you need to add them manually by using `blackbox_register_new_file` once your changes are done.

```bash
# Create the file
touch secrets/demo_file.txt
# ... do your things in the file...
# Encrypt the file
blackbox_register_new_file secrets/demo_file.txt
```

♦️ Note: when you encrypt the file, the file is renamed with a `.gpg` like `secrets/demo_file.txt.gpg` extension. Now the content is not legible (encrypted). When the file is encrypted, you are ready to commit the changes over that file :-)


### Edit a file

You need to decrypt the file first and then re-encrypt the file at the end.

```bash
# Decrypt the file
blackbox_decrypt_file secrets/demo_file.txt
# ... make your changes in the file...
# Re-encrypt the file
blackbox_edit_end secrets/demo_file.txt
```

Once the file is decrypted, it will appear as a new file with the same name without the `.pgp` extension. The file now looks like this `secrets/demo_file.txt` and the content is in clear text (decrypted).

You can perform all the changes you want and then encrypt the file again when you are done. Then the plain file will be automatically removed and the encrypted file updated.

### Delete a file

Use the command `blackbox_deregister_file  <file>`. Afterwards, is very important to run the command `blackbox_shred_all_files` in order to remove decrypted files that could still exist in your machine. (Be aware: there is no typo in the command, the 'a' letter is NOT missed from the `shred` word).

### Decrypt all files

If you need to perform changes in several files, the easiest way is to decrypt all the files at once:

```bash
blackbox_decrypt_all_files
```

Afterwards, when you're done, please manually encrypt all the files, one by one, with the command `blackbox_edit_end secrets/{file_name}`.
There is no option to batch that process yet in Blackbox, but there is a [discussion ongoing about it](https://github.com/StackExchange/blackbox/issues/52).

### Check the differences between a clear file and a encrypted file:

```bash
blackbox_diff secrets/demo_file.txt
```

### :shipit: Using it in CI

One of the coolest features is that you can use pgp to decrypt secrets in any CI process, this is a simple example that
- Load the PGP keys in the machine
- Clone the secret management repository using a github token
- Decrypt one file and store it as `.env`
- Source the `.env` file to the `npm run build` process.

```yml
name: Pull Request Check

on: [pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Ensure Node Version
        uses: actions/setup-node@v3

      - name: import GPG key
        env:
          GPG_KEY: ${{ secrets.GPG_KEY }}
        run: |
          echo $GPG_KEY | base64 --decode > signature.asc
          gpg --batch --import signature.asc
      - name: Clone secrets-management
        env:
          GH_EXTENDED_TOKEN: ${{ secrets.GH_EXTENDED_TOKEN }}
        run: git clone https://${GH_EXTENDED_TOKEN}:x-oauth-basic@github.com/UlisesGascon/super-secrets-management.git

      - name: decrypt secrets (local environment)
        env:
          BRANCH: ${{ steps.vars.outputs.short_ref }}
          GPG_PASSPHRASE: ${{ secrets.GPG_PASSPHRASE }}
        run: gpg --no-tty --batch --passphrase "$GPG_PASSPHRASE" --pinentry-mode loopback --output .env --decrypt super-secrets-management/secrets/app/.env.gpg

      - name: Install dependencies
        run: npm ci

      - name: Build the project
        run: |
          source .env
          npm run build

      #... MORE STEPS
```


### :warning: Important

This is a demo repository, that include me as administrator. If you fork this project, you will inherit me as the admin for your backbox storage. In order to avoid that create the project from zero, following [this guidelines](https://github.com/StackExchange/blackbox#installation-instructions)

