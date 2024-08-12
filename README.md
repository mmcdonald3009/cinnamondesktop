At time of writing I am using Debian 12 which is the current stable release. I am using SpiralLinux which formats as BTRFS and has snapper rollback in GRUB - many many thanks to the OpenSUSE gecko dev for coming up with SpiralLinux.

First - why am I doing this - because as a very experienced senior systems admin who has been fiddling with Linux for years when I saw Cinnamon Desktop with Cinnamenu I knew we finally had a solid windows breaker.
Cinnamon is the maturity KDE wishes it could have. They have retained very early gnome functions which turns out to be the smartest way forward IMHO.
It's also a finger up at the rather obnoxious Linux Mint admin who talked to me like I an an idiot for suggesting my way to disable hidden files for regular users.



What You Get:

1. A self healing Cinnamon implementation using Cinnamenu as default that is stable enough for commercial deployment ( some panel functions like add and remove can get taken out see procedure below ) but MOVE is stil there.
2. When a user toggles hidden files in Nemo, Nemo simply closes and restarts. Not a perfect solution but should keep your office workers from accidentally deleting hidden files.
3. You can set a limit to the number of workspaces a user may create.
4. At anytime you can stop 2 and 3 by typing killall dbus-monitor into a terminal.



The Extra's You Have to Get First...
1. Download Cinnamenu was here at time of writing: https://cinnamon-spices.linuxmint.com/applets/view/322
2. xautomation allows for ESC key emulation when if using EXPO applet to create workspaces ( ctrl + up ) when the count goes above 4, a dconf write to number of workspaces 4 is made, and ESC key triggered to close page.
3. jsonlint checks the integrity of json files.
4. libgtk-3-bin provides gtk-launch for launching .desktop files ( found in /usr/share/applications) from a terminal.
5. So do this: apt install xautomation jsonlint libgtk-3-bin



How It All Works:

1. Copy 11_cinnamon.gschema.override into /usr/share/glib-2.0/schemas. Then as su/sudo run this from a terminal: glib-compile-schemas /usr/share/glib-2.0/schemas/
2. Copy /etc/xdg/autostart/z_login.desktop into /etc/xdg/autostart/. This calls a script (z_login.sh) that runs per user $EUID at login. Very useful and good time to personalise and mount user ID specifics like named home remote mounts/drives/smb shares).
3. mkdir /usr/share/customscripts and copy the files z_login.sh and 9999.json into it. chmod +x z_login.sh.



What Happens At Login Of The Very First User? (/etc/xdg/autostart/z_login.desktop calls script /usr/share/customscripts/z_login.sh). It does the following:

The first user that was created during setup/install and then when logging out and back in for the very first time, a check is made for a file named: ~/.config/firstlogincomplete_DONOTDelete.
This file ~/.config/firstlogincomplete_DONOTDelete won't be there yet, so the Cinnamenu schema file 9999.json wil be copied into ~/.config/cinnamon/spices/Cinnamenu@json, and a dconf write enables the default applets including Cinnamenu.
Now that file ~/.config/firstlogincomplete_DONOTDelete will be created so the check passes over it next time.
You will be logged out just once in this part of z_login.sh: cinnamon-session-quit --logout --force
Now your first ever user when logging in again gets Cinnamenu and won; get logged out again unless ~/.config/firstlogincomplete_DONOTDelete is removed.



What Happens At Login Of Subsequently Created Users?

Subsequent users will get Cinnamenu as default as they were created after we copied 11_cinnamon.gschema.override into /usr/share/glib-2.0/schemas and ran: glib-compile-schemas /usr/share/glib-2.0/schemas/



What Happens At Every Other Login?

At every login we run jsonlint to check the integrity of 9999.json. If the file fails, it will be copied in from and overwritten by /usr/share/customscripts/9999.json




Procedures To Remove Add Panel And Remove Panel From Cinnamon<br>
(First I would take a backup of /usr/share/cinnamon/js/panel.js, main.js and applet.js just in case you need to restore them....)<br>

sed -i 's|menu.addMenuItem(menuItem);||g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
2. sed -i 's|menu.addMenuItem(menu.addPanelItem);||g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
3. sed -i 's|this.addMenuItem(applet_settings_item);||g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
4. sed -i 's|Restart Cinnamon|Restart Display|g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
5. sed -i 's|menu.addMenuItem(panelEditMode);||g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
6. sed -i 's|Restarting Cinnamon|Restarting Display|g' /usr/share/cinnamon/js/ui/main.js 2>/dev/null <br>
7. sed -i 's|this.addMenuItem(new SettingsLauncher(_("System Settings"), "", "preferences-desktop"));||g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
8. sed -i 's|menu.addMenuItem(menu.troubleshootItem);||g' /usr/share/cinnamon/js/ui/panel.js 2>/dev/null <br>
9. sed -i "s|this._applet_context_menu.addMenuItem(this.context_menu_item_remove);||g" /usr/share/cinnamon/js/ui/applet.js 2>/dev/null <br>  
10. sed -i "s|(this.context_menu_item_configure) == -1|(this.context_menu_item_configure) == 100|g" /usr/share/cinnamon/js/ui/applet.js 2>/dev/null <br>  

Finally, please note: I'm not a professional GIT contributor and there may be better ways to present this. Don't criticise me but I am certainly open to good suggestions in polite language, and willing to help with any realistic questions...


END OF README---
