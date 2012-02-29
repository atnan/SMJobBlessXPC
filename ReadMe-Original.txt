Read Me About SMJobBless
====================================
1.1

SMJobBless demonstrates how to embed a privileged helper tool in an application, how to securely install that tool, and how to associate the tool with the application that invokes it.

SMJobBless uses ServiceManagement.framework that was introduced in Mac OS X v10.6 Snow Leopard. 

As of Snow Leopard, this is the preferred method of managing privilege escalation on Mac OS X and should be used instead of earlier approaches such as BetterAuthorizationSample or directly calling AuthorizationExecuteWithPrivileges.


Packing List
------------
The sample contains the following items:

o Read Me About SMJobBless.txt -- This file.
o SMJobBless.xcodeproj -- An Xcode project for the sample.
o Resources -- The project nib.
o MainWindowController.[h,m] -- The sources for SMJobBlessApp. Gets an authorization right to install the privileged helper tool, then calls the ServiceManagement function SMJobBless to do the actual installation.
o SMJobBlessHelper.c -- The source for the minimal helper tool SMJobBlessHelper.
o SMJobBlessHelper-Info.plist -- the property list for the helper tool.
o SMJobBlessHelper-Launchd.plist -- the launchd property list for the helper tool.
o SMJobBlessApp-Info.plist -- the application's property list.

Using the Sample
----------------

1. Run the program.

2. Look in the system log to see the message "SMJobBlessApp[<pid>]: Job is available!". This confirms that the
sample ran correctly.


Building the Sample
-------------------
The sample was built using Xcode 3.2.2 on Mac OS X 10.6.3. 

ServiceManagement.framework uses code signatures to insure that the helper tool is the one expected to be run by
the main application. Therefore, you will need a code signing identity to test this sample. This sample ships with the Code Signing Identity build setting set to "Joe Developer". 

You can get a self-signed code signing identity using these steps:
1. Launch Keychain Access.
2. Select Keychain Access > Certificate Assistant > Create a Certificate...
3. In the Name field, enter "Joe Developer".
4. Change Certificate Type to "Code Signing".
5. Press Continue.
6. You're done!

If you use a signing identity with a different Common Name (CN), you will need to change the CN in four places:

o The Code Signing Identity build setting in the target SMJobBlessApp
o The Code Signing Identity build setting in the target com.apple.bsd.SMJobBlessHelper
o The SMPrivilegedExecutables property in SMJobBlessApp-Info.plist
o The SMAuthorizedClients property in SMJobBlessHelper-Info.plist

Once you have your code signing identity set up and the project configured to refer to the desired code signing identity, you should be able to just choose Build from the Build menu.


How It Works
------------

This sample covers how to install a privileged helper tool that belongs to an application while addressing these challenges:

1. Preserving the ability to drag-install the application.
2. Operating under the principle of least privilege by isolating privlieged code in a separate process instead
of having the entire application running with elevated privileges.
3. Avoiding the use of setuid binaries.
4. Requiring the user to authorize the privileged helper tool only once the first time it's used.
5. Ensuring that the tool hasn't been replaced by another potentially malicious tool.
6. Ensuring that the tool hasn't been co-opted by a different potentially malicious application.

Items 1 and 2 are addressed by shipping the helper tool inside the application bundle without requiring any special ownership or permissions.

Items 3 and 4 are addressed by using launchd to run the helper tool as a daemon with root privileges.

Items 4, 5 and 6 are handled by the SMJobBless function. See the comments in -awakeFromNib and ServiceManagement/ServiceManagement.h for details.

BASIC PROJECT LAYOUT
There are two targets: SMJobBlessApp and com.apple.bsd.SMJobBlessHelper. The application target depends on the helper target. Once the helper is built, it is copied into the application's Contents/Library/LaunchServices directory. Both targets are signed by the "Joe Developer" identity. 

PROPERTY LISTS
The application target has a standard Info.plist associated with it. This plist has an additional key, SMPrivilegedExecutables, whose value is a dictionary. Each key in this dictionary is the name of a privileged helper tool contained in the application. The example has one key, "com.apple.bsd.SMJobBlessHelper". This key maps to a code signing identity, specified as a set of code signing requirements. This is the identity of the tool. This is so that the application can be assured that it is installing the correct tool and not a malicious one that has been put in of it. The example requirement is

identifier com.apple.bsd.SMJobBlessHelper and certificate leaf[subject.CN] = "Joe Developer"

This states that the helper tool's bundle identifier is "com.apple.bsd.SMJobBlessHelper", and it was signed by a certificate whose "Subject Common Name" field is "Joe Developer". If a certificate is obtained through a certificate authority, that authority will not issue more than one certificate for "Joe Developer", so this is a reliable check provided there is also a requirement that the root certificate which signed the leaf certificate is part of the trusted system roots. If this check is not present, any certificate (even a self-signed one) can be made to pass this check.

More information about the code signing requirements language is available at

http://developer.apple.com/mac/library/documentation/Security/Conceptual/CodeSigningGuide/RequirementLang/RequirementLang.html

The helper tool, symmetrically, also specifies a code signing requirement for the applications that are allowed to install it in its Info.plist with an array whose key is SMAuthorizedClients. The example has two entries in the array:

identifier com.apple.bsd.SMJobBlessApp and certificate leaf[subject.CN] = "Joe Developer"
identifier com.apple.bsd.SMJobBlessApp and certificate leaf[subject.CN] = "Joe Developer Development"

These requirements state that tool allows itself to be manipulated by an application that is signed with either Joe Developer's production or development certificates.

The helper tool also has a launchd.plist associated with it. For these helper tools, Program and ProgramArguments are not specified. The system will fill these properties into the plist when it installs the helper, since the tools are placed in a predetermined location -- currently /Library/PrivilegedHelperTools. So this example has only the Label property. Other launchd.plist keys are valid to include; see launchd.plist(5).

The application's Info.plist is distributed in the usual way, by being placed in the bundle. The helper tool's plists must be embedded in the executable itself. This is accomplished by setting special linker flags that create two new sections in the binary: __TEXT,__info_plist and __TEXT,__launchd_plist, for the Info.plist and launchd.plist, respectively. These sections are filled in with the bytes contained in the plist files at link-time. See the helper tool's build settings, under "Other Linker Flags" for the specific arguments used.

The name of the executable produced by the helper target is "com.apple.bsd.SMJobBlessHelper". This name must be the same as the Label attribute in the launchd.plist, which in turn must be the same as the key in the application's SMPrivilegedExecutables dictionary.

This sample as it stands does not actually run the helper tool. The following samples show how to a launchd job and
set up interprocess communication:

o ssd
o BetterAuthorizationSample


Version History
---------------------------
If you find any problems with this sample, please file a bug against it.

<http://developer.apple.com/bugreporter/>

1.1 (Jun 2010) Minor stylistic revisions.
1.0 (Jun 2010) New sample.

Apple Developer Technical Support
Core OS/Hardware

9 Jun 2010
