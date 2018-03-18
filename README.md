# Debit

A simple script which wraps round another simple script to create simple packages.

If you've got a load of files you can download via HTTP(S), you can use this to create a bunch of .deb 
packages to distribute them.

## Debosh

This wraps around Komarov's excellent `debosh`:

    https://github.com/komarov/debosh

specifically, my tweaked fork of it (which I'll try to push upstream if this turns out to be useful to me)

    https://github.com/BigRedS/debosh

Debosh takes a specifically-crafted directory hierarchy and makes a Debian package out of it.

`debit` reads its own config file, crafts a series of these, and uses debosh to create the packages.

## Installation

In simple:

* Get Debosh working (see its own README) then check this repo out. 
* Create a config file at `./debit.cf` and then run `./debit`.

# Configuration

The config file is a yaml file, here's mine:

    # The path to the debosh binary:
    debosh: /home/avi/dev/debosh/bin/debosh
    # Packages will be put here when built:
    package_dir: ~/packages
    # --no-sign and --quietly-dirty are two of my changes to debosh. They means you don't
    # need a working gpg keypair and your packages don't have `~dirty` in their versions
    debosh_args: --no-sign --quietly-dirty
    # Version-numbers are tracked by creating files in this directory. It needs to be 
    # writeable or creatable:
    cache_dir: ./cache
    # You can unset this and just get everything on stdout/stderr:
    log_file: ./debit.log
    

    # Each package has a 'meta' section. If any of the fields are the same for every package
    # you may as well set a default here:
    defaults:
      maintainer: Avi Greenbury <avi@mycompany.com>
   
    # The packages themselves are a hash, see below.
    packages:
      postfixadmininstaller:
        # see 'versions' below for how this works:
        get_version: usr/sbin/postfixadmin-installer --version
        files:
          https://raw.github.com/BigRedS/postfixadmin-installer/master/postfixadmin-installer:
            dir: /usr/sbin
            name: postfixadmin-installer
        meta:
          description: Installer script for PostfixAdmin
          requires:
            perl: 0

      httptraceroute:
        get_version: usr/bin/httptraceroute --version
        files:
          https://raw.github.com/BigRedS/httptraceroute/master/httptraceroute:
            dir: /usr/bin
            name: httptraceroute
        meta:
          description: Traces HTTP redirects
          requires:
            perl: 0
            libwww-perl: 0

      paleo-vim:
        get_version: grep 'lastedit' etc/vim/vimrc.local | cut -d: -f2 
        files:
          http://avi.co/s/vimfix:
            dir: etc/vim/
            name: vimrc.local
        meta:
          description: Set old vim comfig (pre-vim8)
          requires:
            vim: 0

# Versions

When a package is built, the version is written to a file in the `cache_dir` directory. The next time the package
is processed, it is downloaded, the version determined, and the building only happens if the downloaded-version is
later than the previously-downloaded one. This can be skipped with the `--skip-version-check` option.

The package is first assembled; a temporary directory is created (in /tmp) and then each of the files are downloaded
into a directory hierary matching their final prositions (so `/usr/sbin/postfixadmin-installer` ends up in 
`/tmp/<some dir>/sbin/postfixadmin-installer>`)

The `get_version` command is executed inside this directory, and should produce some text output on whose first line 
is a decimal number representing the version of the package. It's normally simply the packaged-script run with its 
`--version` switch, but arbitrary commands may be run (as in the `paleo-vim` package.

# Command-line Options:

    $ ./debit --help
    
    debit: create simple packages of scripts that can be downloaded via http
    
    Usage:
    
      debit <options>
    
    Options:
    
      --config [path] , -c [path]
    
        specify the path to the config file; default: ./debit.cf
    
      --package [package-name]
    
        only build package named 'package-name'
    
      --skip-version-check
    
        don't check package version; (re)build all packages anyway
    
      --logfile [path]
    
        path to log-file; overrides value in config-file
    
      --quiet
    
        only print package names and errors
    
      --list-packages
    
        list configured packages and exit; build nothing
    
      --help
    
        see this help
    
      --preserve-temp-dir
    
        don't delete the temporary working directory; useful for debugging
    
    
    Set the 'DEBUG' environment variable to '1' to get much more output (also 
    sets the --preserve-temp-dir option).




