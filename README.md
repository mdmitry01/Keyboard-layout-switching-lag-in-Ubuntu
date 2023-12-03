## A patch to fix the lag when switching keyboard layouts in Ubuntu 22.04

There is a time lag between the keypress to switch the keyboard layouts (e.g switching from the Ukrainian layout to the English one) and the moment when the actual layout switch happens. Since I switch quite often, this time lag creates a certain amount of discomfort for me. To fix it, I created a patch for [Mutter](https://gitlab.gnome.org/GNOME/mutter) (window manager for the Gnome desktop environment) 3 years ago. I applied the patch to Ubuntu 20.04, which I was using 3 years ago. The problem is that I hadn't published the patch anywhere back in the day, and now I don't remember exactly how this fix works, but it has been working quite well for me for the past 3 years. Today I installed Ubuntu 22.04 on my PC and found out that this layout switching bug still exists in this version of Ubuntu. So I had to go through the process of patching Mutter again. So I've decided to document this patching process, so that next time I don't waste much time on it.

## Patch

First things first, here is the patch

```diff
--- mutter-42.9.orig/src/backends/x11/meta-backend-x11.c
+++ mutter-42.9/src/backends/x11/meta-backend-x11.c
@@ -377,22 +377,6 @@ handle_host_xevent (MetaBackend *backend
             case XkbMapNotify:
               keymap_changed (backend);
               break;
-            case XkbStateNotify:
-              if (xkb_ev->state.changed & XkbGroupLockMask)
-                {
-                  int layout_group;
-                  gboolean layout_group_changed;
-
-                  layout_group = xkb_ev->state.locked_group;
-                  layout_group_changed =
-                    (int) priv->keymap_layout_group != layout_group;
-                  priv->keymap_layout_group = layout_group;
-
-                  if (layout_group_changed)
-                    meta_backend_notify_keymap_layout_group_changed (backend,
-                                                                     layout_group);
-                }
-              break;
             default:
               break;
             }
```

As I said previously, I made this patch 3 years ago and unfortunately, I don't remember exactly how this fix works.

## Download the patched version of libmutter

[Go to the releases page](https://github.com/mdmitry01/Keyboard-layout-switching-lag-in-Ubuntu/releases).

## How to install

1. `sudo dpkg -i libmutter-10-0_42.9-0ubuntu5_amd64.deb`
2. Press alt+f2, type `r` and press Enter to reload Gnome
3. `sudo apt-mark hold libmutter-10-0`. This prevents the libmutter package from being upgraded to the original Ubuntu version

## How to apply the patch and build Mutter

1. `sudo apt build-dep mutter`
2. `sudo apt install build-essential fakeroot devscripts`
3. `mkdir mutter && cd mutter && apt source mutter`
4. `cd mutter-42.9/`
5. Apply the patch
6. `dpkg-source --commit`
7. `DEB_BUILD_OPTIONS=nocheck debuild -us -uc -i -I`. We use `DEB_BUILD_OPTIONS=nocheck` to disable the tests since they fail on my machine for some reason even for the unpatched version of Mutter
8. `cd .. && sudo dpkg -i libmutter-10-0_42.9-0ubuntu5_amd64.deb`
9. `sudo apt-mark hold libmutter-10-0`

See more details at https://help.ubuntu.com/community/UpdatingADeb

## How to install the original unpatched version of libmutter

**Warning**: I haven't tested the command below, but it should work.

```bash
sudo apt install --reinstall libmutter-10-0=42.9-0ubuntu5
```
