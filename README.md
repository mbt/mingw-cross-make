# MingW-w64 Cross Toolchain Builder

Just type `make` in the directory to build it all.

Your user account **must** have `sudo` privileges, and you'll need to
enter your user account's password periodically (during the installation
steps).

You can eliminate this requirement by running the following:

    make PREFIX=${HOME}/opt SUDO=''

This will install things into your home directory and tell the build to
not use sudo.

## Issue: Too Many Open Files

Before running the build, you should run the following command:

    ulimit -n 4096

This will increase the limit of maximum open files.

