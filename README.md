# apt-bundle

Apt-bundle is a simple tool to install a list of packages from a file using the apt(8) package manager on Debian/Ubuntu based systems.  It is useful to set up a fresh machine or a container within a CI environment.  By employing apt-bundle, you can ensure consistent package sets across different environments and seamlessly share development or runtime environments with collaborators.

## Installation

Just copy the `apt-bundle` script to a directory in your `$PATH` and make it executable.

### Use from GitHub Actions

You can use the apt-bundle action to install packages in a GitHub Actions workflow.  Here is an example workflow that installs a list of packages from a Debfile in the repository.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # Assume the source tree has a Debfile
      - uses: actions/checkout@v4

      - uses: knu/apt-bundle@v1
        with:
          debfile: path/to/Debfile  # optional; "Debfile" by default
      # ...
```

#### Inputs

- `debfile`

    The path to the Debfile.  The default is `Debfile` in the repository root.

## Command Usage

`apt-bundle [-n] [-v] [<file>...]`

- `-n`

    Dry-run mode.  The command will not install any packages, but will show what would be done.

- `-v`

    Verbose mode.  The command will show the commands it runs.

- `<file>`

    The file to read the package list from.  The default is `Debfile` in the current directory.

## Debfile Format

Create a text file and list the packages, source lists and keyrings you want in your environment  in that file using the commands described later.  This file is interpreted by `/bin/sh` (dash) as a shell script, and typically named `Debfile`.  Apt-bundle looks for the file with that name in the current directory by default.

```sh
package build-essential
package libreadline-dev
package postgresql-client

# The latest version of git from ppa
ppa git-core/ppa
package git

# Google Cloud SDK from the official third-party repository
keyring cloud.google https://packages.cloud.google.com/apt/doc/apt-key.gpg
source google-cloud-sdk <<EOF
deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main
EOF
package google-cloud-sdk
```

In this file, variables defined in `/etc/os-release` are available so you can use them to conditionally install packages or include values like `$VERSION_CODENAME` in source list definitions.

## Special commands available in Debfile

### package

Usage: `package <name>` | `package <URL>`

This command names a single package name or URL to a .deb file to be installed, which is the only argument to the command.

e.g.
```sh
package faketime
```

### ppa

Usage: `ppa <user>/<name>`

This command adds a Personal Package Archive (PPA) to the system.  The only argument to the command is the name of the PPA in the format `user/ppa`.

e.g.
```sh
ppa git-core/ppa
```

### keyring

Usage: `keyring <name> <URL>` | `keyring <name> <<SH...SH`

This command adds a keyring to the system.  The first argument is the name of the keyring, and the second argument is the URL to the GnuPG public key file.  Instead of specifying the second argument, a shell script that outputs a public GnuPG key or a keyring file to stdout can be fed to the command using a here document.  The keyring file will be saved in `/usr/share/keyrings` with the name `<name>.gpg`.

e.g.
```sh
keyring cloud.google <<SH
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg
SH
```

### source

Usage: `source <name> <<EOF...EOF`

This command adds a source list to the system.  The first argument is the name of the source list and a shell script that outputs the source list should be fed to the command using a here document.  The source list file will be saved in `/etc/apt/sources.list.d` with the name `<name>.list`.

e.g.
```sh
source google-cloud-sdk <<EOF
deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main
EOF
```

## License

Copyright (c) 2024 Akinori Musha

This software is released under the 2-clause BSD license.  See the LICENSE file for details.

Visit [GitHub Repository](https://github.com/knu/apt-bundle) for the latest information.
