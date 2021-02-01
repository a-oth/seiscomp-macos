# SeisComP for macOS port

This is the ported version of SeisComP for Mac OS X / macOS.

The original software has been developed by the [GEOFON Program](http://geofon.gfz-potsdam.de)
at [Helmholtz Centre Potsdam, GFZ German Research Centre for Geosciences](http://www.gfz-potsdam.de)
and [gempa GmbH](http://www.gempa.de). More information can be found on the 
[SeisComP homepage](http://www.seiscomp.de) and on the SeisComP [GitHub repository](https://github.com/SeisComP).

## License

SeisComP is primarily released under the AGPL 3.0. Please check the [license agreement](doc/base/license.rst).

## Checkout the repositories

As noted in the original SeisComP GitHub repository, the software collection is
distributed among several repositories (all forked here). Please check the README in the
original SeisComP repository (included here as README-ORIGINAL) on how to combine the
sources from the different repositories. There you can find an example script that you
can adapt to the seiscomp-macos case here. Note that for marking the difference to the
original repositories, they have been slightly renamed (e.g. main --> seiscomp-main-macos).
However, in order to build SeisComP, they need to be assembled with the naming as in the
original version, so when setting up the script to clone the repositories, you should take
this into account.

## A quick tutorial to compile SeisComP on macOS

seiscomp-macOS installs on:

- macOS High Sierra 10.13.x
- macOS Mojave 10.14.x
- macOS Catalina 10.15.x
- macOS Big Sur 11.x

After compilation seiscomp-macOS will be installed in your Home Directory: ${HOME}/seiscomp

## Pre-Requirements

- Apple Xcode or the Xcode command-line tools, install with:
```
  xcode-select --install
```
- Homebrew package manager for macOS

### Install Apple\'s Xcode

You can choose to install the developer tools via command-line only (saves space),
or download Xcode IDE (8GB or more) from Apple\'s App Store for free.

The command-line works pretty well, so open up `Terminal.app` from Applications > Utilities and type:
 `xcode-select --install`

### Install Homebrew
Homebrew is the missing package manager for Mac: [Homebrew Webpage](http://brew.sh)

In Terminal.app copy/paste this command:

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"`

All the dependencies packages will be installed in /usr/local/Cellar/
	
Check if your system is correctly setup with:
`brew doctor`

Note: If you have Macports package manager installed it is better to not to mix up with Homebrew.
or you could rename Macports default directory /opt/local to /opt/local.OFF

### Install seiscomp dependencies

With Homebrew installed, install seiscomp dependencies packages for macOS:
```
brew install gcc
brew install cmake
brew install flex
brew install coreutils
```

Check installed version of OpenSSL with:
```
brew list openssl
```
It should point to OpenSSL v1.11 and later:
```
/usr/local/Cellar/openssl@1.1/1.1.1d/bin/openssl
/usr/local/Cellar/openssl@1.1/1.1.1d/include/openssl/ (104 files)
/usr/local/Cellar/openssl@1.1/1.1.1d/lib/libcrypto.1.1.dylib
/usr/local/Cellar/openssl@1.1/1.1.1d/lib/libssl.1.1.dylib
....
```

If an older version is installed (like OpenSSL 1.0), then delete this version
```
brew uninstall openssl
```

Then (re)install latest version of OpenSSL with Homebrew:
```
brew install openssl
```

Install Boost:
```
brew install boost
```

Until recently there were compilation issues with boost version >= 1.73, while version
1.72 worked fine. This has been resolved now, but we included the possibility to pass a
version variable `BOOST_VERSION_REQUIRED` with cmake in case the user has installed a
specific boost version as boost@version. Please note that seiscomp does not compile using
the easily available version boost@1.60. You can use the `brew extract` command to extract
a given version of boost into a private tap and then install it, for instance boost@1.72.
Note that you should in that case not forget to link boost@1.72 so that the link
/usr/local/opt/boost@1.72 exists, since cmake is looking for that.

### Install Qt5 for the GUIs

Note: macOS 10.13 and later is required for Qt5.
`brew install qt`


If you need Qt4 for any reason (macOS < 10.13 ): Qt4 is no longer officialy supported by Homebrew 
[see homebrew-qt4 page](https://github.com/cartr/homebrew-qt4).
```
brew install qt@4
```
If this does not work then try with command:
```
brew tap cartr/qt4
brew tap-pin cartr/qt4
brew install qt@4
```

### Install MySQL
The latest Homebrew version installs MySQL8 by default, which used to have some
upgrade issues with Seiscomp3 InnoDB, but this has been fixed recently. You can also use
mysql@5.7, that should also work fine.

```
brew install mysql
```

Note: for compilation with MySQL 8, you might possibly need to change the following line
in the file usr/local/Cellar/mysql/8.[VERSION]/include/mysql/mysql_com.h:

```
#include <mysql/udf_registration_types.h>
```

to

```
#include "mysql/udf_registration_types.h"
```

since otherwise you get compilation issues \"mysql/udf_registration_types.h file not found with angled\".


### Configure MySQL at startup

Edit the default MYSQL configuration file that should be located in /usr/local/etc/my.cnf.
For better performance with a MySQL database, adjust the following parameters:

```
innodb_buffer_pool_size = 64M
innodb_flush_log_at_trx_commit = 2
```

To have launchd start mysql now and restart at login:
`brew services start mysql`

Recommended: MySQL Workbench application for macOS.
MySQL Workbench from Oracle is a free GUI to administer MySQL databases.
[Install MySQL Workbench from Oracle\'s website](https://dev.mysql.com/downloads/workbench/)

### Build seiscomp for macOS

Having combined all the repositories needed as mentioned above, create the installation
directory ${HOME}/seiscomp and the build-directory (e.g. builds-seiscomp-macos), like this:

Create the installation directory in your Home folder:
```
mkdir -p ${HOME}/seiscomp
```

Create the build-directory:

```
mkdir ~/Downloads/seiscomp-git
mkdir ~/Downloads/seiscomp-git/builds-seiscomp-macos
```

Then clone the seiscomp repositories into ~/Downloads/seiscomp-git folder, so that you get
a subfolder like seiscomp ~/Downloads/seiscomp-git/seiscomp-macos-master.
Now we are ready to compile seiscomp-macos with GUI manually from the builds directory, like this:

```
cd builds-seiscomp-macos
cmake -DCMAKE_PREFIX_PATH=/usr/local/opt/qt/ -DCMAKE_INSTALL_PREFIX=${HOME}/seiscomp -DSC_GLOBAL_GUI_QT5=ON ../seiscomp-macos-master
make -j 2
make install
```

For QT4, you need to pass `SC_GLOBAL_GUI=ON` rather than `SC_GLOBAL_GUI_QT5=ON`.

Note: If you have some specific version of Python required that you would like to use, you can
either adapt the CMakeLists.txt file to set the necessary paths accordingly or pass the variable
`Python_VERSION_REQUIRED` to use with cmake, with the version number you would like to use:

```
cmake -DCMAKE_PREFIX_PATH=/usr/local/opt/qt/ -DCMAKE_INSTALL_PREFIX=${HOME}/seiscomp -DSC_GLOBAL_GUI_QT5=ON -DPython_VERSION_REQUIRED=[VERSION] ../seiscomp-macos-master`
```

If everything compiled fine, the files will be installed in ${HOME}/seiscomp.

 
### Increase max open files for seedlink on macOS system startup

To avoid getting seedlink errors when starting seiscomp with \"files open exceed max files ...\",
increase the max open files on system\'s startup.

Create a plist file named: `limit.seiscomp3-maxfiles.plist` with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>limit.seiscomp3-maxfiles</string>
<key>ProgramArguments</key>
<array>
     <string>launchctl</string>
     <string>limit</string>
     <string>maxfiles</string>
     <string>524288</string>
     <string>524288</string>
</array>
   <key>RunAtLoad</key>
   <true/>
   <key>ServiceIPC</key>
   <false/>
 </dict>
</plist>
```

Place this plist file in /Library/LaunchDaemons/

Then set root:wheel permission with command:
`sudo chown root:wheel /Library/LaunchDaemons/limit.seiscomp3-maxfiles.plist`

Launch it with command:
`sudo launchctl load -w /Library/LaunchDaemons/limit.seiscomp3-maxfiles.plist`

### A note on the GUIs for macOS

On macOS, you need to deactivate the App Nap functionality for the GUIs to operate without
crashes and peer inactivity errors. To do that easily systemwide, you can use the OnyX
software (tested with version 3.8.5 on macOS Catalina) to easily do that (tab Parameters
--> Misc --> Turn off App Nap). Alternatively, you can do that via the command line or
check out to deactivate App Nap for individual apps.

### Check SeisComP webpage for the documentation and help
http://www.seiscomp.de

### Troubleshooting

If you get the -lcrypto compile error:
`ld error: -lcrypto not found` 

then you need to fix it. Just do a:
`brew doctor`
which warns you that /usr/bin occurs befor /usr/local/bin:

`Warning: /usr/bin occurs before /usr/local/bin`

You need to edit your $PATH in ~/.bashrc accordingly so that /usr/local/bin occurs before /usr/bin.

Then in the Terminal, source your ~/.bashrc file:
`source ~/.bashrc`

