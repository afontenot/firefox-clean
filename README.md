**This project has been archived! It became too difficult to maintain as a single person trying to patch a very large project. I have moved to [LibreWolf](https://librewolf.net/) which maintains a similar patch set, and recommend others do as well.**

# firefox-clean
A PKGBUILD for Firefox with defaults for the privacy-conscious and power users

**Important note as of Firefox 93.0:**

The patches here currently do not disable the ads Mozilla has built into the URL bar results. These results will appear if you're in the U.S., even if you have searching from the URL bar disabled. You can disable this, so long as Mozilla honors the setting, by setting `browser.urlbar.suggest.quicksuggest.sponsored` to false.

I do not know when this anti-feature will be patched out. I could, of course, just ship a user.js that disables it, but the whole premise of this project is that you shouldn't have to trust Mozilla to do the right thing and honor the setting. In fact you *shouldn't* trust them to do the right thing, as they have repeatedly shown in the past that they are not trustworthy about issues like this.

The difficulty is that the tendrils of this new anti-feature run very deep, and it's hard for someone without deep knowledge of the structure of the Firefox code to 100% eliminate all this bad code and keep it maintained. 

I am growing increasingly frustrated by this project and extremely tempted to move to a different browser entirely. Unfortunately most of the Firefox soft-forks just follow the user.js strategy. If anyone out there has the requisite expertise to help me patch Firefox, please file an issue.

Changes 
 * Completely disables Pocket
 * Completely removes Mozilla's new [remotely pushed hidden extensions](https://blog.mozilla.org/data/2018/08/20/effectively-measuring-search-in-firefox/) (Telemetry Coverage, Search Telemetry, etc). These anti-features don't respect Firefox's telemetry settings.
 * Removes advertisements from the New Tab page and settings for "recommendations"
 * Add option to restart browser to file menu (currently not possible for WebExtensions)
 * Give the user the ability to disable Mozilla addon signing requirement by setting an about:config option
 * Use DuckDuckGo by default
 * Set Google's Safe Browsing off by default (can be enabled if desired)

 Please file a bug report to request additional changes

 Don't trust me! Diff this source tree with the original Archlinux package here: [https://git.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/firefox](https://git.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/firefox)

## userChrome.css

I've included my userChrome.css that includes several other changes I think are worthwhile. These are *not* part of the firefox-clean patch set, and won't be built into your browser. A future goal of this project is to (re-)enable desirable configurability to Firefox, and features that currently require userChrome.css are a good starting place.

## Other Projects

If you're not an Arch Linux user, consider using this project to create a custom Firefox profile that disables many anti-user features: [https://github.com/allo-/firefox-profilemaker](https://github.com/allo-/firefox-profilemaker).
