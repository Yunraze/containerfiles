# nvim Toolbox Container

My objective was to create a container that I could use at work, and at home for the express purpose of using the same set of tools everywhere. I wanted simple configurability, and a "DevContainer" feel to it, meaning it contains my own configuration and all the software, but the work files are mounted as a volume. And all of this in a way that is compatible with SELinux.

## Getting started

**NOTE:** You can skip the `keys` subdirectory and remove those sections from the `Containerfile` if you don't need SSH keys, for e.g. GitHub, or GPG keys, for signing or encryption.

**NOTE:** If you do use SSH keys or GPG keys, never EVER use them without a password. Storing them into a container __IS NOT SECURE__ without a password. Even then your private keys are in a container. Do not push this kind of an image into a public registry. Use separate keys for toolbox containers that you can easily revoke.

The image is based on Arch Linux `(archlinux:latest)`. Some extra software is installed, and configured with the accompanying `packages.txt`. It is assumed that the `keys` subdirectory contains `gpg` and `ssh` subdirectories for both types of keys respectively. The directory structure looks like this:

```
nvim-toolbox-archlinux/
├── Containerfile
├── keys
│   ├── gpg
│   │   ├── private_keys.txt
│   │   ├── public_keys.txt
│   │   ├── public.asc
│   │   ├── private.asc
│   │   ├── some_other_public.asc
│   │   └── trustlevel.txt
│   └── ssh
│       ├── id_rsa
│       └── id_rsa.pub
├── packages.txt
└── README.md

```

* `Containerfile`: If you want to remove SSH key or GPG key processing, just comment it out or delete it here.
* `keys/gpg/private_keys.txt`: List of the GPG private keys (in the same directory) *and their passwords*, separated by a space, that you want copied over. One key and password per line. **Password protect your private keys!**
* `keys/gpg/public_keys.txt`: List of the GPG public keys (in the same directory) that you want copied over. Just the key file names.
* `keys/gpg/trustlevel.txt`: List of assigned trustvalues for your public keys. Export this for your keys with `gpg --export-ownertrust`, remove the commented lines and save it as a text file. If you don't do this, `git log --show-signature` will complain about the trust of the keys.
* `keys/ssh/*`: Put your private and public key files here. **Password protect your private keys!**
* `packages.txt`: List of packages to install to the container.
* `README.md`: You are here.

### Handling GPG keys

#### Start by finding the key(s) id you want to migrate by using this command:

```sh
gpg --list-secret-keys --keyid-format LONG
```

It should returns something like:

```
sec   rsa4096/[**YOUR KEY ID**] 2024-03-30 [SC]
      ABCDEFGHIJKLMNOPQRSTUVWXYZ
uid                 [ultimate] username (KEY NAME) <user@domain>
ssb   rsa4096/ABCDEFGHIJKL 2024-03-30 [E]
```

After the key size rsa4096/ is your key ID.

#### Export the public key

```sh
gpg --export -a [your key id] > your-public-key.asc
```

#### Export the private key (if password protected, you’ll be prompted to enter it)

```sh
gpg --export-secret-keys -a [your key id] > your-private-key.asc
```

## Configure the environment (Dotfiles)

All of the configuration should be done via your personal Dotfiles. This includes things like Neovim configuration, `.bashrc`, `.bash_profile` and anything else you'd keep in your home directory.

## Building the container

The container is designed to adapt to your configuration. In practice this means that you have your own `$HOME` directory with all the configuration that you want, and the actual work happens elsewhere. In my configuration that is the `/workspace` directory. You need to supply a few arguments to the build:

* `USERNAME`, defaults to `dev`. This is the name of the user account inside the container that you want to use. It doesn't need to match the username on your host system.
* `DOTFILES_REPO`, defaults to mine. You want to create a bare git repository and store your configuration files there.
* `BRANCH`, defaults to `nvim-toolbox-arch`. This is the branch in the repository you want to use. I use separate branches for each of the devices I use.

Build your container like so (replace the name with whatever you want, and use your own dotfiles):

```sh
podman build --build-arg USERNAME=dev \
             --build-arg DOTFILES_REPO=https://github.com/Yunraze/Dotfiles.git \
             --build-arg BRANCH=nvim-toolbox-arch \
             -t localhost/nvim-toolbox .
```

## Run the container

Remove `--rm` if you don't want the container to be throwaway. Customize the `-v` volume mount to point to the directory you want to mount into the container's workspace. The `:Z` is there to make SELinux happy. Remove it if you're not running SELinux. `--userns=keep-id:uid=1000,gid=1000` is needed to keep user permissions inside the container. *Make sure the `uid` and `gid` match the user and group ID you want to use.*

```sh
podman run --rm -it -v $(pwd):/workspace:Z --userns=keep-id:uid=1000,gid=1000 nvim-toolbox
```

## Share or move the container to another machine

You can use `podman save` and `podman load` to share the image when it isn't available locally or remotely, or can't be built.

Save the image to a tar archive with:

```sh
podman save --output toolbox.tar nvim-toolbox-arch
```

Recreate an image from the tar archive with:

```sh
podman load --input toolbox.tar
```
