Legacy IONDRVSupport
====================

This repository contains patched IONDRVSupport.kext for macOS 10.5, which correctly handles pixel format when video drivers without acceleration are used (e.g. QEMU).

Issue presence can be observed by crashes of various applications (e.g. VNC servers) or by running the sample tool:

```c
// gcc t.c -framework CoreFoundation -framework ApplicationServices -o t
#include <CoreFoundation/CoreFoundation.h>
#include <ApplicationServices/ApplicationServices.h>
#include <stdio.h>

int main(void) {
  CGDirectDisplayID d = CGMainDisplayID();
  unsigned int bpp = CGDisplayBitsPerPixel(d);
  // Prints 24 or 32 under normal conditions.
  printf("CGDisplayBitsPerPixel -- %u\n", bpp);
  unsigned int bps = CGDisplayBitsPerSample(d);
  // Prints 8 under normal conditions and garbage with stock IONDRVSupport (e.g. 530)
  printf("CGDisplayBitsPerSample -- %u\n", bps);
  return 0;
}
```

Patched IONDRVSupport.kext can be downloaded from [releases](https://github.com/acidanthera/IONDRVSupport/releases) and injected with OpenCore:

```xml
<dict>
  <key>Arch</key>
  <string>i386</string>
  <key>BundlePath</key>
  <string>IONDRVSupport.kext</string>
  <key>Comment</key>
  <string>Acidanthera IONDRVSupport</string>
  <key>Enabled</key>
  <true/>
  <key>ExecutablePath</key>
  <string>IONDRVSupport</string>
  <key>MaxKernel</key>
  <string>9.99.99</string>
  <key>MinKernel</key>
  <string>9.0.0</string>
  <key>PlistPath</key>
  <string>Info.plist</string>
</dict>
```

Force-loading IOGraphicsFamily is generally required for patched IONDRVSupport.kext injection.
