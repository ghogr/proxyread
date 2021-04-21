# A bash script that reads proxy settings from GNOME or KDE.

This tool reads proxy configuration from desktop environments such as GNOME or KDE, and simply returns it as-is, for use in command line arguments. This is written to avoid hard-coding proxy configuration in `.desktop` files as much as possible.

## Usage

Install the tool to your desired path and use it in things like your `.bashrc`, `.zshrc`, or `.desktop` files.
Note that for `.desktop` files, the __full__ path to the binary is required.

## Example use cases

### 1. `.bashrc` and `.zshrc`
You may find these useful to have in your `.bashrc` and `.zshrc`:
```bash
export http_proxy="$(pxr -p http)"
export ftp_proxy="$(pxr -p ftp)"
export https_proxy="$(pxr -p https)"
```

### 2. `.desktop` files for Flatpak applications

Flatpak applications generally do not respect proxy settings from the host. Assuming Flatpak applications are installed system-wide, you can find all exported files installed by Flatpak in `/var/lib/flatpak/exports/share/applications/`.
  
Since these are symlinks, you need to follow the symlinks and copy files you wish to override to `$HOME/.local/share/applications/`.

The following two examples are focused on Chromium/Electron applications, as they frequently will need unrestricted internet access to function. The general idea here is to modify the `Exec` directive and insert the proxy arg immediately after the Flatpak Application ID. This only facilitates basic internet access, features such as WebRTC will still remain broken, since Chromium will not use proxies for WebRTC.

- `org.chromium.Chromium.desktop`: 
    ```
    Exec=/usr/bin/flatpak run --branch=stable --arch=x86_64 --command=/app/bin/chromium  --file-forwarding org.chromium.Chromium --proxy-server="$(/usr/local/bin/pxr -p socks -s 5)" @@u %U @@
    ```

- `com.simplenote.Simplenote.desktop`:
    ```
    Exec=/usr/bin/flatpak run --branch=stable --arch=x86_64 --command=simplenote --file-forwarding com.simplenote.Simplenote --proxy-server="$(/usr/local/bin/pxr -p socks -s 5)" @@u %U @@
    ```

Additionally, it is recommended to change the `GenericName` entries in your `.desktop` files to something more distinguishable, so you can be sure you are launching the modified `.desktop` file.

After you have modified your `.desktop` files, remember to update the desktop menu database. On GNOME 40, it should just show up automatically in your list of applications (try `xdg-desktop-menu forceupdate` if it does not), and you need to run `kbuildsycoca5` on KDE.