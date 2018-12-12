# firefox-clean
A PKGBUILD for Firefox with defaults for the privacy-conscious and power users

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

# Other Projects

If you're not an Arch Linux user, consider using this project to create a custom Firefox profile that disables many anti-user features: [https://github.com/allo-/firefox-profilemaker](https://github.com/allo-/firefox-profilemaker).
