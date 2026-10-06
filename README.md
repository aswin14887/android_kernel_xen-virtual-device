# android_kernel_xen-virtual-device
Kleaf build files for the Xen guest kernel (`xen_virtual_device_aarch64`) that runs Android 17 as a Xen PV
domU on the Raspberry Pi 5.

## Patches

`patches/` holds changes to the kernel tree (`common/`) that this build needs, one folder per change with the
patch and a README explaining why and how to apply it.

| Folder | Change |
|---|---|
| `xen-drm-front-abgr8888/` | Xen PV display frontend accepts XBGR8888/ABGR8888, the format of SurfaceFlinger's client target. |
