Introduction
------------
OpenPasswordFilter is an open source custom password filter DLL and userspace service to better protect / control Active Directory domain passwords.

The genesis of this idea comes from conducting many penetration tests where organizationas have users who choose common passwords
and the ultimate difficulty of controlling this behavior.  The fact is that any domain of size will have some user who chose
`Password1` or `Summer2015` or `Company123` as their password.  Any intruder or low-privilege user who can guess or obtain
usernames for the domain can easily run through these very common passwords and start expanding the level of access in the 
domain.

Microsoft provides a wonderful feature in Active Directory, which is the ability to create a custom password filter DLL.  This
DLL is loaded by LSASS on boot (if configured), and will be queried for each new password users attempt to set.  The DLL simply
replies with a `TRUE` or `FALSE`, as appropriate, to indicate that the password passes or fails the test.  

There are some commercial options, but they are usually in the "call for pricing" category, and that makes it a little 
prohibitive for some organizations to implement truly effective preventive controls for this class of very common bad passwords.  

This is where OpenPasswordFilter comes in -- an open source solution to add basic dictionary-based rejection of common passwords.

OPF is comprised of two main parts:

   1. OpenPasswordFilter.dll -- this is a custom password filter DLL that can be loaded by LSASS to vet incoming password changes.
   2. OPFService.exe -- this is a C#-based service binary that provides a local user-space service for maintaining the dictionary and servicing requests.
  
The DLL communicates with the service on the loopback network interface to check passwords against the configured database
of forbidden values.  This architecture is selected because it is difficult to reload the DLL after boot, and administrators
are likely loathe to reboot their DCs when they want to add another forbidden password to the list.  Just bear in mind how this
architecture works so you understand what's going on.

**NOTE** The current version is very ALPHA!  I have tested it on some of my DCs, but your mileage may vary and you may wish to test in a safe location before using this in real life.

Installation
------------
You can download a precompiled 64-bit version of OPF from the following link:

[OPF-alpha.zip](https://github.com/jephthai/OpenPasswordFilter/raw/master/OPF-alpha.zip)

For this to work at all, you must have complexity requirements enabled.  This is in the local security policy for the 
domain controller -- here is some documentation from Microsoft for enabling it:

  https://technet.microsoft.com/en-us/library/Cc786468(v=WS.10).aspx

You will want to configure the DLL so that Windows will load it for filtering passwords.  Note that you will have to do this
on all domain controllers, as any of them may end up servicing a password change request.  Here is a link to Microsoft's
documentation for setting up a password filter:

    https://msdn.microsoft.com/en-us/library/windows/desktop/ms721766(v=vs.85).aspx
    
The bottom line is this:

  1. Copy `OpenPasswordFilter.dll` to `%WINDIR%\System32`
  2. Configure the `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Notification Packages` registry key with the DLL name
  
Note, you do not include the `.dll` extension in the registry key -- just `OpenPasswordFilter`.

Next, you will want to configure the OPF service.  You can do so as follows (suppose you have installed the files in `C:\OPF`):

    > sc create OPF binPath= c:\opf\opfservice.exe start= boot

Finally, create a dictionary file in the same directory where `opfservice.exe` lives named `opfdict.txt`.  This should contain
one forbidden password per line, such as:

    Password1
    Password2
    Company123
    Summer15
    Summer2015
    ...

If all has gone well, reboot your DC and test by using the normal GUI password reset function to choose a password that is on
your forbidden list.


