# `SMJobBless` + XPC #

`SMJobBlessXPC` is based off Apple's `SMJobBless` sample code, but uses XPC over Mach ports for communication between the app and privileged helper tool.

## Original README ##

For the original `README` that ships with Apple's `SMJobBless` sample code, see `ReadMe-Original.txt`.

## Outline of changes to the original sample code ##

The following changes have been made to Apple's sample code:

* Project files and build configuration have been updated to more modern settings.
* Issues that were causing compile warnings have been corrected.
* Code signing identity has been changed from "Joe Developer" to "Mac Developer". You will need to edit `SMJobBlessApp/SMJobBlessApp-Info.plist` and `SMJobBlessHelper/SMJobBlessHelper-Info.plist` to reflect the Common Name (CN) of the certificate you're using to sign your app & helper tool.
* The helper tool registers a Mach service named `com.apple.bsd.SMJobBlessHelper` with `launchd`.
* The host application connects to the helper tool using XPC via the registered Mach service, sends a message and receives a response.
* A status log is displayed in a text field in the host application to show what's going on.