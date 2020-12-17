
## MIFF build - with patches


Win10 build setup

When installing, the following workloads must be checked:

* “Desktop development with C++” (under the Windows group)

* “Game development with C++” (under the Mobile & Gaming group)

In addition, go to the Individual Components tab and make sure the following components are selected under the “SDKs, libraries, and frameworks” group:

* “Windows 10 SDK” (at least version 10.0.17134.0)

* “C++ ATL for v142 build tools (x86 and x64)” (also select ARM64 if you’ll be building for ARM64)



Check out mercurial

Mozilla Firefox release tags are here:

https://hg.mozilla.org/releases/mozilla-release/tags

Pick one (we are currently on FIREFOX_80_0_BUILD2 /
FIREFOX_80_0_RELEASE tags), pull changeset hash):

hg clone -u bd5d1f49975deb730064a16b3079edb53c4a5f84 https://hg.mozilla.org/releases/mozilla-release/

Goes into m041/mozilla-release

cd mozilla-release
./mach bootstrap
./mach build
./mach run


The setup_patch_environment script takes as an argument the directory of the Firefox source code. The patches are edited to point to the given Firefox directory and copied into a new directory, and the Quilt file .pc/.quilt_patches is edited to point to this new directory. The Firefox source code directory must be within m041.


For branding images, copy the folders in branding/ into the Mozilla source code directory. The file trees match Mozilla's source code, so the new icons will overwrite the existing brand images in the source tree.


Resources:

https://firefox-source-docs.mozilla.org/setup/windows_build.html#building-firefox-on-windows

https://firefox-source-docs.mozilla.org/contributing/vcs/mercurial.html

