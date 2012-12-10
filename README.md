# CUBRID RDBMS port for Mac OS X

In this tutorial we will go through the entire process of **building CUBRID CCI API on Mac OS X**. Along the way we will patch CUBRID for Mac OS X. Even though these changes may not be enough to build CUBRID on Mac, it is enough to build CUBRID CCI driver for Mac.

There are some small differences between building on Snow Leopard, Lion, or Mountain Lion in the way that some system header files are located in different locations. I will cover all the steps below.

## Requirements

### Install Xcode

Apple's [Xcode](https://developer.apple.com/technologies/tools/) Developer Tools version 4.1 or later for Lion, 3.2 or later for Snow Leopard, or 3.1 or later for Leopard is required to build Macports (see below). You can install Xcode from Mac App Store or download it from [Apple Developer Center](https://developer.apple.com/xcode/).

When installing ensure that the **optional components** for command line development are checked to be installed ("*System Tools*", "*UNIX Development*", or "*Command Line Tools*" in newer versions, or "*Command Line Support*" in older ones). The following is a screenshot of Xcode 3.2.6 on Snow Leopard.

![Figure 1: Installing Xcode and optional components on Snow Leopard.](http://www.cubrid.org/files/attach/images/194379/022/389/xcode_optional_components.png)

### Install Macports

[Macports](http://www.macports.org/) is a great, convenient package manager for Mac OS X. It simplifies many steps by automatically managing dependencies between packages. Some Linux-only packages have been ported to Mac and are available through Macports. To build CUBRID we need some of those GNU packages available on Linux. So, we need Macports.

Macports is available in **.pkg** installer which you can download from [http://www.macports.org/install.php/](http://www.macports.org/install.php/). Once downloaded, simply double click on the package to start the installation process.

### Install Dev. Tools

Now install the tools required to build CUBRID.

    sudo port install autoconf automake libtool coreutils

## Prepare CUBRID Source

### Checkout SVN source code

[CUBRID Source Code](http://www.cubrid.org/wiki_tutorials/entry/cubrid-source-code) is available at SF.net. Checkout the latest version using the following command. In this tutorial we will download version 8.4.1.

    svn co http://cubrid.svn.sourceforge.net/svnroot/cubrid/cubrid/branches/RB-8.4.1

The above will download **RB.8.4.1**  branch, esectially the 8.4.1 branch, into **RB.8.4.1** directory.

### Create symlinks to GNU libs

We need to create symlinks to two GNU libs (*these are those Dev. Tools*) because Mac OS X version of these libs are not compatible with CUBRID source code as we use GNU version. So let's link them.

    cd RB-8.4.1
    ln -s /opt/local/bin/greadlink readlink
    ln -s /opt/local/bin/glibtoolize libtoolize

These will create **readlink** and **libtoolize** symlinks in the root directory of CUBRID source code (*RB-8.4.1*).

Now we need to force the builder to use these new GNU libraries instead of the Mac OS X version by appending the **current directory** (inside *RB-8.4.1*) path to `$PATH`. So, the below will be different for you depending on where you have extracted CUBRID source code.

    export PATH=/Users/user/Downloads/RB-8.4.1/:$PATH
    $PATH
    -bash: /Users/user/Downloads/RB-8.4.1/:/opt/local/bin:/opt/local/sbin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/local/git/bin:/usr/X11/bin: No such file or directory

To find the current directory path you can execute `pwd` command:

    $ pwd
    /Users/user/Downloads/RB-8.4.1

### Configure files

Some files in CUBRID source code need to be executable while for some reason they are not. So we will fix this.

    cd RB-8.4.1
    chmod +x external/libregex38a/configure
    chmod +x external/libregex38a/install-sh

For **CUBRID 8.4.3** the following new files (*absent in previous versions*) also need to be executable.

    cd RB-8.4.3
    chmod +x external/expat-2.0.1/configure

## Apply Mac OS X specific Patch

In this repo you will find the patch for CUBRID 8.4.1, 8.4.3, and 9.0 in the following files:

1. *rb-8.4.1-svn-diff.patch*
2. *rb-8.4.3-svn-diff.patch*
3. *rb-9.0.0-svn-diff.patch*

These files need to be copied into the directory where you have extracted CUBRID source code. In our case it is **RB-8.4.1/**.

To apply *rb-8.4.1-svn-diff.patch*, for example, on RB-8.4.1 directory, run the following command.

    cd RB-8.4.1
    # rb-8.4.1-svn-diff.patch should already be here
    patch -p0 < rb-8.4.1-svn-diff.patch

If you want to run a **demo patch** first without actually applying the patch, use `--dry-run` option after the `patch` command as follows:

    patch -p0 < rb-8.4.1-svn-diff.patch --dry-run

This will simply output which files would have been changed if the patch was actually applied.

## Build

Once you hav applied the patch, it is time to configure and make the build.

### Run autogen

    cd RB-8.4.1
    mkdir build
    ./autogen.sh

### Configure the build

Now we are ready to configure the build. After we configure and make the build, we can either have 64 bit version of CUBRID or its drivers or 32 bit version. Depending on what you need, you can add or omit `--enable-64bit` option. In the following example, I will build 64 bit version.

    cd build
    ../configure --enable-64bit

If you are building **CUBRID 8.4.3** or **9.0**, before moving to the next final step, you need to **make** external libraries which are required for CUBRID CCI driver. In 8.4.1 this step is not required.

    cd build/external
    make

### Make CUBRID CCI driver

    cd build/cci
    make -j

If everything went fine, you can find the compiled CUBRID CCI libraries in ***RB-8.4.1/build/cci/.lib/***. Notice that ***.lib*** directory is hidden because it preceeds with a period. You may not see it in the Finder. You have to use your Terminal to see hidden directories and files.

Now install the libraries. Notice that you need **sudo** privileges.

    sudo make install

Depending on your local environment and which `make` program is used, the libraries and headers may end up in different places. The `make install` command will output the target location where it did install the files, so you can go and check yourself.

#### Installed files

In my case, the files were installed to */Users/user/cubrid/lib* and */Users/user/cubrid/include* directories. Most common and proper location is */usr/lib* and */usr/include* directories.

The following files were installed to the target location:

1. Libraries
	1. */Users/user/cubrid/lib/libcascci.8.dylib*
	2. */Users/user/cubrid/lib/libcascci.a*
	3. */Users/user/cubrid/libcascci.dylib* (which is a symlink for *libcascci.8.dylib*)
	4. */Users/user/cubrid/libcascci.la*
2. Header files
	1. */Users/user/cubrid/include/cas_error.h*
	2. */Users/user/cubrid/include/cas_cci.h*

### If you need to install other CUBRID libraries

If you plan to link to this CUBRID CCI library from other CUBRID drivers such as PHP, Python, etc., you need to create one more symlink. If you want to directly work with CCI library, you can skip this step.

    sudo ln -s /Users/user/cubrid/lib/libcascci.8.dylib /Users/user/cubrid/lib/libcascci.so

We need this symlink because CUBRID drivers try to dynamically link to **.so** library which is a common library extension on Linux. On Mac OS X it is **.dylib**. But as of 8.4.3 CUBRID drivers link to **.so**, so we need to create this symlink.

Done! Now you can follow the following tutorials to install other CUBRID drivers which have dynamic dependency on CUBRID CCI driver.

- [Build CUBRID PHP Driver on Mac OS X through PECL using CCI Driver](http://www.cubrid.org/wiki_apis/entry/build-cubrid-php-driver-on-mac-os-x-through-pecl-using-cci-driver)
- [Build CUBRID PHP Driver on Mac OS X using CCI Driver](http://www.cubrid.org/wiki_apis/entry/build-cubrid-php-driver-on-mac-os-x-using-cci-driver)
- [Build CUBRID PHP Driver for XAMPP on Mac OS X using CCI Driver](http://www.cubrid.org/wiki_apis/entry/build-cubrid-php-driver-for-xampp-on-mac-os-x-using-cci-driver)

If you have questions, feel free to tweet me [@CUBRID](http://twitter.com/cubrid) or ask your question at [CUBRID QA& site](http://www.cubrid.org/questions).