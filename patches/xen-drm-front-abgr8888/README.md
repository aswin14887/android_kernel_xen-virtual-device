# drm/xen-front: accept XBGR8888 and ABGR8888

`0001-drm-xen-front-accept-XBGR8888-ABGR8888.patch` applies to the kernel tree at `common/` (android17-6.18).
It adds two pixel formats to the plane of the Xen PV display frontend (`drivers/gpu/drm/xen/xen_drm_front_conn.c`).

## Why

The Android guest (`xen_sdv_ivi`) shows its display on the Raspberry Pi 5's HDMI panel next to the HAR cluster
through a Xen PV display (`vdispl`):
* guest `drm/xen-front`
* dom0 `xen-vdispl-be`
* HAR overlay plane

Without this patch the guest shows nothing.

* SurfaceFlinger always allocates its client target as `HAL_PIXEL_FORMAT_RGBA_8888`
  (`frameworks/native/services/surfaceflinger/CompositionEngine/src/RenderSurface.cpp`). In DRM terms that is
  `DRM_FORMAT_ABGR8888`. No property changes it: `ro.surface_flinger.default_composition_pixel_format` does not
  apply to this buffer.
* The stock frontend lists XRGB/ARGB8888 but not XBGR/ABGR8888. drm_hwcomposer's `AddFB2` therefore fails DRM
  core's "some plane supports this format" check with `-EINVAL` (`could not create drm fb -22` in logcat). Every
  frame is rejected (`Failed to create AtomicCommitArgs for frame composition`), and SurfaceFlinger reports
  `BAD_DISPLAY`.

The format is passed to the backend in `XENDISPL_OP_FB_ATTACH`, so the protocol is unchanged. `xen-vdispl-be`
handles all four 32-bit formats.

## Apply and build

```sh
cd <android_kernel>/common
git apply ../common-modules/xen-virtual-device/patches/xen-drm-front-abgr8888/0001-drm-xen-front-accept-XBGR8888-ABGR8888.patch
cd ..
tools/bazel run //common-modules/xen-virtual-device:xen_virtual_device_aarch64_dist
```

The result is `out/xen_virtual_device_aarch64/dist/Image`. Copy it to wherever the Android product takes its
guest kernel prebuilt from, and to dom0's `/home/root/android/Image`.

## Verified

Built and booted on 2026-10-06 (Raspberry Pi 5, Xen 4.19). The guest's composer then committed frames to the Xen
display: 46 fps during the boot animation, as counted by `xen-vdispl-be`. CarLauncher showed on the HDMI panel
next to the cluster.

## Status

Local to this tree; not sent upstream. The change is generic: any Android guest on `drm/xen-front` hits the same
problem. It is a candidate for upstream Linux or for ACK (Android Common Kernel).
