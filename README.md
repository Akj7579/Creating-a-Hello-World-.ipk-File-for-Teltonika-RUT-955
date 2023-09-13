# Creating a Hello World .ipk File for Teltonika RUT 955

In this guide, we'll walk you through the steps to create a simple "Hello, World!" .ipk package for the Teltonika RUT 955 device using OpenWrt.

## Clone OpenWrt from GitHub

To get started, clone the OpenWrt repository from GitHub:


    git clone https://git.openwrt.org/openwrt/openwrt.git

Update and Install 'Feeds' Packages

Navigate to your OpenWrt directory and update and install the 'feeds' packages:


	cd openwrt
	./scripts/feeds update -a
	./scripts/feeds install -a

Configure the Cross-Compilation Toolchain

Invoke the graphical configuration menu:


make menuconfig

From the menu, select the following options:

'Atheros ATH79' as Target System,'Generic' as Subtarget,'Teltonika RUT955' as Target Profile

After making these choices, select 'Exit' from the configuration menu and save your changes. You can now build the target-independent tools and the cross-compilation toolchain:



    make toolchain/install

The target-independent tools and the toolchain will be deployed to the staging_dir/host/ and staging_dir/toolchain/ directories. This applies to the executables built in the above section as well as the pre-built executables available in the SDK.



    export PATH=/home/buildbot/source/staging_dir

## Create the Hello World Source Code

Create a C source code file named helloworld.c with the following content:



    #include <stdio.h>
    
    int main(void)
    {
        printf("\nHello, world!\n\n");
        return 0;
    }

## Create the Package Structure

Navigate to your OpenWrt directory and create the package structure:


    mkdir -p package/mypackages/examples/helloworld
    cd package/mypackages/examples/helloworld
    touch Makefile

Create a Makefile with the following content:



    include $(TOPDIR)/rules.mk

    # Name, version, and release number
    PKG_NAME:=helloworld
    PKG_VERSION:=1.0
    PKG_RELEASE:=1

    # Source settings
    SOURCE_DIR:=/home/buildbot/helloworld

    include $(INCLUDE_DIR)/package.mk

    # Package definition
    define Package/helloworld
      SECTION:=examples
      CATEGORY:=Examples
      TITLE:=Hello, World!
    endef

    # Package description
    define Package/helloworld/description
      A simple "Hello, world!" application.
    endef

    # Package preparation instructions
    define Build/Prepare
        mkdir -p $(PKG_BUILD_DIR)
        cp $(SOURCE_DIR)/* $(PKG_BUILD_DIR)
        $(Build/Patch)
    endef

    # Package build instructions
    define Build/Compile
        $(TARGET_CC) $(TARGET_CFLAGS) -o $(PKG_BUILD_DIR)/helloworld.o -c $(PKG_BUILD_DIR)/helloworld.c
        $(TARGET_CC) $(TARGET_LDFLAGS) -o $(PKG_BUILD_DIR)/$1 $(PKG_BUILD_DIR)/helloworld.o
    endef

    # Package install instructions
    define Package/helloworld/install
        $(INSTALL_DIR) $(1)/usr/bin
        $(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/usr/bin
    endef

    # This command is always the last, it uses the definitions and variables above
    $(eval $(call BuildPackage,helloworld))



## Create feeds.conf

Navigate to your OpenWrt directory and create a feeds.conf file:



    cd /home/../source
    touch feeds.conf

Modify the feeds.conf file to specify the local package feed:


‮
## Update the Package Feed

In your OpenWrt directory, update the package index and make your package available in the configuration menu:


    ./scripts/feeds update mypackages
    ./scripts/feeds install -a -p mypackages

If the last step completes successfully, you should see the response: "Installing package 'helloworld' from mypackages."
Compile Your Package

Compile the 'helloworld' package:



    make package/helloworld/compile



This will create helloworld.ipk.

Congratulations! You've successfully created a "Hello, World!" .ipk package for the Teltonika RUT 955 device using OpenWrt.

