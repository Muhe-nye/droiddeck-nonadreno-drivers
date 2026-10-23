# Steam black-screen workaround for the Venus route

The Mali setting already disallows Steam GPU web compositing, but
`DeviceSupport.openDriverActive()` returns false when Venus is enabled. The
Venus launch therefore missed `BL_STEAM_SOFT_COMPOSITING=1` and still passed
`-cef-force-gpu` without `-cef-disable-gpu-compositing`.

This focused patch makes the existing software-web-compositing path the default
when `BL_VENUS=1`. It leaves the game's Vulkan/OpenGL driver route and the
non-Venus default unchanged. Apply it to
`Droid-Deck/DroidDeck@9ae771c50d575fa94de93c577e5d8bc653011a68`.

The five `test_steam_presentation.py` cases execute the production launch code
with a recording client. They pass locally, including the new Venus-default and
explicit-software-compositing cases.

On a REDMI K80 Ultra (Mali G925), three starts with the experimental native
buffer-release patch alone remained black. Software web compositing reached the
full Steam interface both with that native patch and with the original native
module restored. This separates the observed black-screen workaround from the
independent native buffer-release corrections.

Device testing uses the existing app's supported `droiddeck-env` configuration
to keep `BL_STEAM_SOFT_COMPOSITING=1` across starts. No APK uninstall or user-data
reset is required. The source patch is intended to remove that manual setting
from future builds.

For the tested package (`com.droiddeck.launcher`), the supported persistent file
is `/sdcard/Android/data/com.droiddeck.launcher/files/droiddeck-env`. Add the line
`BL_STEAM_SOFT_COMPOSITING=1` while retaining any other settings, then restart
the DD session. Updating Steam itself did not remove this setting in the device
retest. A future app version or a different package may require rechecking the
configuration path. Removing this one line reverts the workaround after a restart.

This is a launch-policy workaround for the DD test build, not a replacement
PanVK binary. The tested renderer was Venus over the phone's proprietary Mali
driver, even though PanVK bundles were installed and selected in the saved
driver preferences. Direct PanVK with GPU web compositing has not yet been
validated on this device.

Current limitation: the tested Steam WebHelper reports that this flag disables
browser GPU acceleration, so Steam menu animation may be less smooth. It does
not select software rendering for games. In observed Kingdom Rush gameplay,
the compositor reported approximately 59–61 presented frames per second over
consecutive ten-second windows. This is a compositor statistic, not a full
game-engine benchmark or a promise for other games.
