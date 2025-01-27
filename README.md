# MiFF

MiFF[^1] is lightway approach to replace Mozilla and Google (and any
other) service dependencies from Mozilla Firefox (GeckoView),
including removing anything resembling "phone home", as well as some
modifications to certain settings. Note that we "replace" service
dependencies, we don't "remove" them.

This project was partly inspired by the [ungoogled-chromium
project](https://github.com/Eloston/ungoogled-chromium) and borrows
from their methodology.

We host latest builds on ``cdn.privacy.app``, updates are signed and
provided as well (once you've installed MiFF, updates should "just
work").

* [Windows (v100.x)](https://cdn.privacy.app/miffrelease/MiFF-100.0.2.1.exe)
* [MacOS (v89.x)](https://cdn.privacy.app/miffrelease/MiFF-89.0.0.1.en-US.mac.dmg)
* [Linux (v100.x)](https://cdn.privacy.app/miffrelease/MiFF-100.0.2.1.tar.bz2)

You can download/install the above and just use them straight off the bat.
They should (hopefully) give you a more private browsing experience than
vanilla Firefox.[^1a]

The rest of this README documents what we're trying to accomplish with
this project, as well as open sources all of our modifications and
backend server code.

# Table of Contents

<details open>
<summary><i>Click to collapse/expand ToC</i></summary>

<br/>

<!-- MarkdownTOC -->

1. [Introduction](#introduction)
1. [Building MiFF Yourself](#building-miff-yourself)
1. [Patches](#patches)
1. [Platform-specific Builds](platform-specific-builds)
    1. [Windows (Win10/11) Build Setup](#windows-win10-build-setup)
    1. [Ubuntu Build Setup](#ubuntu-build-setup)
    1. [macOS Build Setup](#set-up-on-mac-os-x-m16)
	1. [Mozilla Build Process](#check-out-with-mercurial)

<!-- /MarkdownTOC -->

</details>


## Older MiFF versions

Source code releases are [tagged on
github](https://github.com/Magnusson-Institute/miff/tags). Not all
versions are supported for all targets. Expand below for details.  We
track Firefox releases [shortly after Mozilla releases
them](https://www.mozilla.org/en-US/firefox/releases/).[^1b]

<!-- TODO: add system requirements -->

<details>
<summary><b>Click to expand/collapse</b></summary>
<!-- MarkdownMiffVersions -->

### v100.0.2.1

Windows installer: https://cdn.privacy.app/miffrelease/MiFF-100.0.2.1.exe

Linux: https://cdn.privacy.app/miffrelease/MiFF-100.0.2.1.tar.bz2

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v100.0.2.1

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/100.0.2.1/releasenotes/


### v98.0.1.1

Windows installer: https://cdn.privacy.app/miffrelease/MiFF-98.0.1.1.exe

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v98.0.1.1

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/98.0.1/releasenotes/


### v96.0.1.1

Windows installer: https://cdn.privacy.app/miffrelease/MiFF-96.0.1.1.exe

Linux: https://cdn.privacy.app/miffrelease/MiFF-96.0.1.1.tar.bz2

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v96.0.1.1

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/96.0.1/releasenotes/


### v92.0.0.1

Windows installer: https://cdn.privacy.app/miffrelease/MiFF-92.0.0.1.exe

Linux: https://cdn.privacy.app/miffrelease/MiFF-92.0.0.1.tar.bz2

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v92.0.0.1

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/92.0/releasenotes/


### v89.0.2.3

MacOS installer: https://cdn.privacy.app/miffrelease/MiFF-89.0.0.1.en-US.mac.dmg
(note: this is v89.0.0.1).

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v89.0.2.3

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/89.0.2/releasenotes/

Earlier point releases for v89.0.2.x:

* https://github.com/Magnusson-Institute/miff/releases/tag/v89.0.2.2
* https://github.com/Magnusson-Institute/miff/releases/tag/v89.0.2.1

### v89.0.0.1

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/89.0/releasenotes/

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v89.0.0.1

### v84.0.2.4

Source: https://github.com/Magnusson-Institute/miff/releases/tag/v89.0.2.4

Matching Firefox release notes: https://www.mozilla.org/en-US/firefox/84.0.2/releasenotes/

<!-- MarkdownMiffVersions -->
</details>





<a id="introduction"></a>

# Introduction

This project is built from Mozilla's open source GeckoView software.
This project is not affiliated with Mozilla Foundation, Mozilla Corporation, Google, or Alphabet.
If you make and/or distribute your own build of miff or miff-backend, please
respect Mozilla's guidelines.[^2]

MiFF is open source and distributed under MPL 2
(https://www.mozilla.org/en-US/MPL/2.0/), see the "LICENSE" file for
details.

# Building MiFF Yourself

<!-- TODO: add reference to the UK paper that runs browsers for 30 minutes; include our objective
to have zero internet accesses that were not initiated by user. -->

<a id="patches"></a>

Regardless of what platform you are building for, there's a common flow:

1. Set up your build system
    1. Set up _general_ tooling
    1. Set up _mozilla specific_ tooling
    1. Set up _platform specific_ tooling and/or tweaks
1. Build "latest and greatest" Firefox (Nightly)
    1. Download Nightly from Mozilla version control (mercurial) servers
    1. Build and run
1. Build _specific_ release of Firefox
    1. Download specific version (tarball) of Firefox (e.g. v92.0)
    1. Build and run
1. Build _patched_ version of _specific_ release
    1. Download (matching) specific version of MiFF (this git, pick matching tag)
    1. Apply patches
    1. Build and run

So, yes, if you're doing this the first time on a build system, you
will end up building three times ... first "Nightly", then a specific
release version of Firefox, then finally a patched version of that
specific release, and that's "MiFF". The reason for this approach is
to safeguard against tooling issues. Trust us, you do _not_ want to be
debugging that.

There are two main parts to "MiFF", more or less: first are a set of
patches, as noted above, which modifies behavior. Second is a
stand-alone backend server.

All of MiFF's patches for the Gecko source code are located in the
``patches``:

* All patches follow naming conventions of ``NN_<description>.diff``.
* The ``patches/series`` file lists all patches, and the order
  in which they will be applied. By naming convention, they are
  applied in "NN" order.
  
Contents of the diff files follow the unified GNU diff format.[^3]

*Note: for many of the patch files, we include extensive comments on
all of the changes int he patchfiles themselves.*

## Branches

master - active development; all releases (v92.x, v096.x, etc) are on the master

stable - mirror of tagged releases (in case you need to try building on new targets for example)



## MiFF patches / changes

There are two sources of changes:

* File patches, these are encompassed by the `miff/patches/*.diff`
  files, and managed with `quilt`.

* Replacement files.  These are listed in `miff/copy_files/` and are
  copied over with `copy_files.sh` into the firefox source tree.

If you're just applying changes and patches and re-building, do
something like this:

```bash
cd /c/mozilla-source/firefox-84.0.2
../miff/copy_files.sh
ln -s ../miff/patches .
quilt push -a
./mach build
./mach run
```


## Working with the update patch (patch #12)

If you have not run ``./mach build`` before, quilt will fail trying
to apply 12_updates.diff. The build process creates several generated
files on a first run, including the certificates for update validation.
You will need to run ``./mach build`` first, then apply patch 12 and
beyond.

There is an additional step if you are not working in a Windows
environment. The initial build creates an obj-\* folder, where all the
generated files live. The name of this folder is dependent on the OS.
For non-Windows systems, create a symbolic link to your platform's
obj-\* folder named ``obj-x86_64-pc-mingw32`` and the patch will
apply correctly. The relevant files in this folder are ``primaryCert.h``
and ``secondaryCert.h``.

Updates are delivered through MAR (Mozilla ARchive) files. The MAR files
are signed by an RSA private key with a corresponding signed certificate.

The public key data is contained in the .h files listed above, represented
in ``0x**`` hexadecimal octets. The following commands can be used to
properly convert a PKCS7 certificate to hexadecimal:

```
bash
openssl pkcs7 -print_certs -in CERTIFICATE.p7b -outform PEM -out CERTIFICATE.pem
openssl x509 -inform PEM -in CERTIFICATE.pem -outform DER -out CERTIFICATE.cer
xxd -i CERTIFICATE.cer
```


## Working with the release patch (patch #99)

The final patch in the series is used to disable debug features and to
track the version number. If you are working on development you will want
to leave this patch unapplied. Before creating a release/update, set the
appropriate version number in this patch and create a matching tag on Github.

These features are controlled by the mozconfig files, one for each file.
The mozilla build tool will only use the mozconfig if the build is run like
so: ``env MOZCONFIG="path/to/mozconfig" ./mach build``.

Any changes to mozconfig or the version number trigger a full build.

And you should have a working, re-branded Firefox.


## Making modifications yourself

First make sure you've done the above steps. 'miff' needs to be
alongside your build directory, you need a symbolic link to 'patches',
etc.

For example, if you want to start making changes to 'aboutDialog.ftl'.
First, apply patches and file replacements as per above. Then:


```
bash
cd /mozilla-source/firefox-84.0.2
quilt new NN_description_of_changes.diff
quilt add browser/locales/en-US/browser/aboutDialog.ftl 
```


Where 'NN' is a new (higher) patch number than what is already in
`miff/patches/series`. Quilt will only track changes made *after* a file is added to a patch.

Now make some edits to this file (aboutDialog.ftl). Then refresh the patch file:

```
quilt refresh
```


That will create an 'NN' patch file.

## To work with an existing patch / set of changes


You will need to selectively 'quilt push' until you are at the patch
file you want to be using to cluster your changes.  Make sure the
file(s) you are working with are referenced in that patch file (if not
add them with `quilt add <filename>`.

## Some principles

* Try labeling changes with the "MIFF NN" string
  where 'NN' is the patch (diff) file
  (it will be unique, does not exist in FF source code outside dictionary files)
  (note: older tags might use "MagIns")

* Try not just deleting or replacing things, but comment out the
  old code, so that when continuing to work with the resulting
  modified files, you can see what's been done (roughly)


Expand section below for details on all the current patch files.

<details>
<summary><b>Click to expand/collapse</b></summary>

<a id="patch-01"></a>

### ``01_privacy``

* Makes several changes to baseline configuration (towards more private)

	* Disables all telemetry and remove related UI elements
	
	* Removes pings for Top Sites, Snippets, addon recommendations, etc.

* Disables Mozilla's default browser agent (Windows only)


### ``02_sso``

* Changes Mozilla FxA (Firefox Accounts) endpoints to Privacy.App endpoints
  <!-- TODO: generalize this to ``miff-backend`` -->
* Removes email verification from FxA login process

* Changes Mozilla HAWK requests to XHR, thus using Flask session for authentication

* Uses "Magnusson Institute Member" as a generic username for FxA



### ``03_sync``

* Similar to ``02_sso`` but changing requests for sync

* Takes out cryptographic wrappers based on username/password combinations


### ``04_connected_devices``

* Disables features that depend on device push

	* Synced Tabs
	
	* "Connect Another Device" feature
  

### ``05_search``

* Removes Google, Bing, and Amazon as default search engines

* Sets DuckDuckGo as the replacement default search engine

	* Startpage is added as a search engine option during installer creation


### ``06_ui``

* Removes default Mozilla bookmarks

* Removes recommended sites and pinned search engines from Home screen

* Disables custom profile pictures for FxA

* Removes the about:mozilla page

* Removes unneeded URL parameters for Privacy.App endpoints


### ``07_pocket``

* Removes "Recommended by Pocket" sponsored articles

* Removes Save to Pocket feature


### ``08_endpoints``

* Routes Safebrowsing and Blocklist requests through Privacy.App
  <!-- TODO: generalize this to ``miff-backend`` -->

* Routes update checks for Addons and GeckoMediaPlugins through Privacy.App
  <!-- TODO: generalize this to ``miff-backend`` -->

* Routes downloads for Addons through Privacy.App
  <!-- TODO: generalize this to ``miff-backend`` -->


### ``09_support_links``

* Change help and support links to Privacy.App
  <!-- TODO: generalize this to ``miff-backend`` -->



### ``10_branding_text``

* Changing brand names to comply with Mozilla Trademark Guidelines[^2]
  <!-- TODO: generalize this to ``miff-backend`` -->


### ``11_various_branding``

* Changing brandind messages to comply with Mozilla Trademark Guidelines[^2]
  <!-- TODO: generalize this to ``miff-backend`` -->




### ``12_updates``

* Change update server to be from Privacy.App
  <!-- TODO: generalize this to ``miff-backend`` -->



### ``13_permissions``

* Some fixes to support web apps that require local storage
  but do not work with cookies - in particular so that
  ``snackabra`` (https://snackabra.io) and similar apps can work properly
  even with privacy settings dialed up



### ``99_disable_debug``

This patch file is a bit special. It turns off browser debugger and
other debugging tools. This is necessary from the workflow of
developing for ``miff``, since we debug patches using these
tools, but a shipping final binary shouldn't have them enabled.

</details>





## Platform-specific Builds

Below are sections on Windows, Linux, and MacOS. Expand the one you need.

<details>
<summary><i>Windows Build</i></summary>

### Windows (Win10) Build Setup

When installing, the following workloads must be checked:

* “Desktop development with C++” (under the Windows group)

* “Game development with C++” (under the Mobile & Gaming group)

In addition, go to the Individual Components tab and make sure the
following components are selected under the “SDKs, libraries, and
frameworks” group:

* “Windows 10 SDK” (at least version 10.0.17134.0)

* “C++ ATL for v142 build tools (x86 and x64)” (also select ARM64 if
  you’ll be building for ARM64)


### Set up Cygwin

In Windows, we work in either Moz Shell for all the build tools from Mozilla (see
below), and Cygwin64 for all of our own tooling (git, quilt, various
shellscripts, etc).

Install the following packages in Cygwin:

* git
* quilt
* p7zip

</details>

<details>
<summary><i>Linux (Ubuntu) Build</i></summary>

### Ubuntu build setup

First, install Python (3.6 or later):
``sudo apt install python3 python3-dev python3-pip``

The Firefox documentation recommends downloading Mercurial through pip, but apt works as well. Run either command:

```bash
python3 -m pip install --user mercurial
sudo apt install mercurial
```

You will also need to install yasm and libgtk2.0-dev through ``apt``.

The rest of the process is similar to a Windows setup, but all commands can be done from the Ubuntu terminal.

</details>



<details>
<summary><i>macOS Build</i></summary>

<!-- might need:
export CPATH="/Users/petermagnusson/.mozbuild/macos-sdk/MacOSX10.12.sdk/usr/include"
-->

## Set up on Mac OS X (m1)[^4]

The C++ tools used to build on Mac are based off Xcode; so first
install latest version of Xcode from the App Store, then finalize it's
installation from command line, and install Mercurial. Current macOS
is Python 3.9.7 which works fine.

```
# install basics
brew install mercurial yasm quilt

# stay up-to-date, especially on m1 ...
brew update
brew upgrade
```

<!-- you might need to install libgtk2.0 - 
   https://gitlab.gnome.org/GNOME/gtk-osx/-/blob/master/gtk-osx-setup.sh
   (used to be able to do brew install libgtk2.0-dev ...)
   -->



Next, create a working directory where you want to work, here we'll
call it "~/dev/ff01"; create it and bootstrap:
   
```
mkdir ~/dev/ff01
cd ~/dev/ff01
curl https://hg.mozilla.org/mozilla-central/raw-file/default/python/mozboot/bin/bootstrap.py -O
python3 bootstrap.py
```


Press "enter" for destination, for default; so it'll start in
"~/ff01/mozilla-unified" in this example.  Mercurial will pull from
"https://hg.mozilla.org/mozilla-unified"; which is full tree (be
patient, it needs to download 650,000+ files). We will build that
first, to ensure our tooling etc is properly set up.  Follow
instructions from the script, then make sure to start a new terminal so
all the settings have taken effect.

The various tooling specific to FF build will be set up by the above bootstrap in ``~/.mozbuild/``
	
If you have problems running the bootstrap (eg "no node" errors), you might want to try directly pulling with mercurial:

```
hg clone https://hg.mozilla.org/mozilla-central/ firefox-source
cd firefox-source
```
	
(See https://firefox-source-docs.mozilla.org/contributing/vcs/mercurial.html)

<!--
The following is a bit outdated, it doesn't seem to be needed for
v96.0 onwards, we did need some of this stuff historically, dating
back to v89. Instead, seems that all you need to do now is adjust
some paths.

################################################################
A few more macOS particulars: 

```
sudo xcode-select --switch /Applications/Xcode.app
sudo xcodebuild -license
echo "export PATH=\"$(python3 -m site --user-base)/bin:$PATH\"" >> ~/.zshenv
python3 -m pip install --user mercurial
hg version
```

HOWEVER. Your "latest version" of Xcode will probably have an SDK that
is too modern. So you need to "downgrade" locally for Moz.  At time of
writing, their documentation[^5] states that they are using the 10.12 SDK, but their _error messages_
state that they support the 11.1 SDK.

(Apple documentation on the different versions is summarized here:
https://developer.apple.com/support/xcode/#minimum-requirements ).

The older (documentation) instructions suggests pulling 10.12 SDK from
Xcode 8.2. We will go with that for now. Download:

_(Update: mozbug trackers seem to indicate they're using 12.2 from
8.3.3 now, at https://developer.apple.com/download/all/?q=8.3.3 which will be a 'xip' file)_

``https://download.developer.apple.com/Developer_Tools/Xcode_8.2/Xcode_8.2.xip``

It's big (4.2 GB), unzip and pull out the 10.12 SDK by "opening" the
file - it'll look like an xcode app copy in your Download folder, but
it's "really" directory tree under ~/Downloads/Xcode.app:


```
mkdir -p ~/.mozbuild/macos-sdk
# This assumes that Xcode is in your "Downloads" folder
cp -aH ~/Downloads/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk ~/.mozbuild/macos-sdk/
```


And add the following line to the "mozconfig" file (which will be
created if it's not there); should be in your FF source code
directory:

```
echo "ac_add_options --with-macos-sdk=$HOME/.mozbuild/macos-sdk/MacOSX10.12.sdk" >> ~/dev/ff01/mozilla-unified/mozconfig
```

################################################################
-->

Again, make sure to start a new terminal so all the settings have
taken effect, and then you should be able to start the (huge) build:

```
cd ~/dev/ff01/mozilla-unified
./mach build
./mach run

# if you want to try to package it, you would also:
# ./mach package
```


the object tree will be in:

```
~/dev/ff01/mozilla-unified/obj-x86_64-apple-darwin21.3.0
```


Next, build the same (or very similar) version of FF from a clean
source code tarball. Make sure to match (exactly) the tagged version
in miff (e.g. from top of
``https://github.com/Magnusson-Institute/miff/tags``).

In this case, our latest miff tag at time of writing is "96.0.1.1",
which matches Mozilla FF tag "96.0.1" (the fourth digit ".1" is our
internal release schedule). So in this case, download
``https://archive.mozilla.org/pub/firefox/releases/96.0.1/source/firefox-96.0.1.source.tar.xz``,
download our own (tagged) miff tarball, and place it alongside,
extract all the tarballs, net result should look like:


```
#
# eg in this case you're downloading:
# https://github.com/Magnusson-Institute/miff/archive/refs/tags/v96.0.1.1.tar.gz
# https://archive.mozilla.org/pub/firefox/releases/96.0.1/source/firefox-96.0.1.source.tar.xz
#
# and result should be:
#
~/dev/ff01/mozilla-unified/...
~/dev/ff01/firefox-96.0.1/..
~/dev/ff01/miff-96.0.1.1/...
#
```


First re-build clean 96.0.1 by itself _without_ applying any patches,
to make sure your build environment is all working.

But first adjust your paths so that some key tool binaries are picked up
from "~/.mozbuild" rather than your default macOS tooling.

```
# insert clang and node from mozbuild
export PATH="/Users/<you>/.mozbuild/clang/bin:/Users/<you>/.mozbuild/node/bin:$PATH"

# examples assume this root dev directory
cd ~/dev/ff01

# if you haven't extracted it yet:
tar xzf ~/Downloads/firefox-96.0.1.source.tar.xz

cd firefox-96.0.1

<!--  ## not needed?
# remember to update/create mozconfig:
# (it might not exist)
echo "ac_add_options --with-macos-sdk=$HOME/.mozbuild/macos-sdk/MacOSX10.12.sdk" >> ./mozconfig
-->

# now this should work:
./mach build
./mach run
```

Now you can apply the patches:

 
```
# make sure we're in the right place
cd ~/dev/ff01

# first, even if it's a tarball, needs to be called 'miff':
mv miff-96.0.1.1 miff

# make sure you're in the right spot
cd ~/dev/ff01/firefox-96.0.1

# first copy the files that are meant to outright over-write:
../miff/copy_files.sh

<!-- note - if you're using some old tarballs for miff, you might need alias for 'm041' -->

<!-- is this still needed?
# make sure your actual "obj" directory can be reached from the reference directory:
# (otherwise some patches will break)
ln -s obj-x86_64-apple-darwin21.3.0 obj-x86_64-pc-mingw32
-->

# now soft-link our patch system and apply them
ln -s ../miff/patches .
quilt push -a

# the above might fail on Patch 12, that's ok, first build with patches 1-11:
./mach build
./mach run

# then apply Patches 12+ and build again
quilt push -a
./mach build
./mach run

# and if that all looks good, build a .dmg,
# the result will be in obj-*/dist
./mach package
```


And there we go!

## Catching up

If you've already built a version of MiFF, and just want to catch up
to a newer one, then often you can re-use some of your earlier work.

First, move aside the old MiFF tree, e.g. let's say you're jumping from
84.0.2 to 96.0.1, you would end up doing something like:


```
cd ~/dev/ff01
# move old miff aside
mv miff miff-84.0.2
# extract new firefox version code
tar xzf ~/Downloads/firefox-96.0.1.source.tar.xz
# extract miff patches
tar xzf ~/Downloads/miff-96.0.1.1.tar.gz
# miff needs to be in 'miff'
mv miff-96.0.1.1 miff
```

</details>

<details>
<summary><i>Mozilla Build Process</i></summary>

### Check out with Mercurial

In addition to the above tools, Firefox downloads several necessary toolchains
as part of ``./mach bootstrap``. However, ``./mach bootstrap`` only works with
Firefox source code downloaded through Mercurial bundles, not release tarballs.

To get a working build environment, clone the head of the mozilla-central
Mercurial bundle:

```bash
hg clone https://hg.mozilla.org/mozilla-central/
```

Next, 'bootstrap' all the tools and configurations needed, following
instructions along these lines:

```bash
cd mozilla-central
./mach bootstrap
./mach build
./mach run
```

On Linux environments, ``./mach bootstrap`` will also prompt several ``apt``
package downloads. It can sometimes take multiple bootstrap runs to actually
download all the packages.

The toolchains downloaded during ``./mach bootstrap`` are the *latest*
versions of the tools. If you are working with a Firefox version more than a
few versions behind official releases, there will often be issues building the
tarball. Furthermore, on Windows 'bootstrap' is very dependent on the Visual
Studio installation. Updating Visual Studio tends to break the build command
entirely, and you will have to run 'bootstrap' again (which, if you haven't
pulled from the mozilla-release head recently, will probably lead back to the
first problem).


### Take a specific tarball

Now grab a specific version that we have patch support for. For our examples
here, we will use ``84.0.2`` throughout, but you can see latest tagged releases
on our github at https://github.com/Magnusson-Institute/miff/tags

Visit https://archive.mozilla.org/pub/firefox/releases/84.0.2/source/ and
download the compressed (xz) tar ball.  Untar it alongside mozilla-release and
move the ''miff'' folder right next to it, should eventually get a folder
directory like this:

```bash
mozilla-central/
firefox-84.0.2/
miff/
```

Next, go to the tarball release folder (again, ``84.0.2`` throught our example)
and build it clean. Note that ``./mach bootstrap`` is not run for the tarball.

```bash
cd firefox-84.0.2
./mach build
./mach run
```

To build an existing version of miff, you will need a matching miff "release".
These releases use the Firefox version number as a root, with an added digit for
miff changes. For the example of Firefox version 84.0.2, you would find a
(tagged) miff release like v84.0.2.4 and checkout the tag:

```bash
cd miff/
git checkout v84.0.2.4
```



## Creating an update file

When the Firefox browser updates, the files in a user's install directory are
replaced by new files in the update package. These updates are packaged as a
special type of xz or bz2 archive called a MAR (Mozilla Archive). There are
two tools that are available to create a MAR: a signmar tool created during
the normal build process (obj*/dist/bin/signmar), and a Python tool
(https://github.com/mozilla/build-mar). We need both to create a working
update. The signmar creates a file manifest, but cannot sign the MAR; the
Python tool can sign, but does not generate a file manifest.

The Python tool can be installed with pip, but requires several other
tools in order to install properly.

For Cygwin:

* python38
* python38-devel
* python38-cryptography
* liblzma-devel

For Ubuntu:

* liblzma-dev

</details>

<!--

## NOTE (2021=12=21) on Mac m1

so i don't forget ... looks like their nightly (latest) nowadays can
work fine with the latest SDK (2021-12-22), however, that's not the
case with immediately recent version (e.g. 89.0.2); and looks like one
wants python 3.8 specifically, might need some "hard coding" of setup:


```
   brew reinstall python@3.8
   brew doctor
   brew link --overwrite python@3.8
   which python3
   python3 --version
   brew reinstall hg
   brew link --overwrite mercurial
   hg --version
   brew update
```


might need on second round of build to tell mach that yes system python3 is ok:

```
   export MACH_USE_SYSTEM_PYTHON="yes try it"
```


might run into issues with missing headers, try this (this takes a while):

```
   sudo rm -rf /Library/Developer/CommandLineTools
   xcode-select --install
   cd /Library/Developer/CommandLineTools/Packages/
   open macOS_SDK_headers_for_macOS_10.14.pkg
```


here's a collection of pesky SDKs:

https://github.com/phracker/MacOSX-SDKs/releases

i went with 11.1 instead.


-->


# Resources

https://firefox-source-docs.mozilla.org/setup/windows_build.html#building-firefox-on-windows

https://firefox-source-docs.mozilla.org/contributing/vcs/mercurial.html


# LICENSE

MiFF is open source and distributed under MPL 2
(https://www.mozilla.org/en-US/MPL/2.0/), see the ["LICENSE"](LICENSE) file for details.



----------------

# Footnotes

[^1]: We would call it "Mostly It's Firefox", but that would be in
      violation of Mozilla's (reasonable) trademark rules.[^2]
      And we didn't want to call it "unmozzilad firefox", because
      we're big fans and that's too negative. A more correct name
      might be "ungoogled-firefox" but that would confuse most people.
      And we can't be clever like "GNU" ("Gnu's Not Unix") because
      neither Mozilla nor Firefox starts with a vowel. In short,
      officially, "MiFF" doesn't stand for anything at all.

[^1a]: Currently this does connect to https://privacy.app - issue 4
       (https://github.com/Magnusson-Institute/miff/issues/4) will
       allow control over that.

[^1b]: For an exhaustive set of mozilla tags for Firefox, see
       https://hg.mozilla.org/releases/mozilla-release/tags

[^2]: https://www.mozilla.org/en-US/foundation/trademarks/policy/

[^3]: https://www.gnu.org/software/diffutils/manual/html_node/Detailed-Unified.html

[^4]: Latest succesful build: macOS Monterey (12.2.1), MacBook Pro (16-inch, 2021), Apple M1 Max.

[^5]: https://firefox-source-docs.mozilla.org/setup/macos_build.html#macos-sdk-is-unsupported

	  
