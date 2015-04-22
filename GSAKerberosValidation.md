## Introduction ##

Diagnostics utility to help identify kerberos-related configuration/setup and serving issues for the Google Search Appliance (GSA).

It is assumed the GSA Admin ran through the kerberos IWA setup for the GSA (i.e, created an account, created a DNS name, ran ktpass)

Use this script if you run into issue activating IWA on the GSA or if you get prompted by the GSA while doing a secure search.  This utility will **not** check to see if why certain secure results are not being shown (see our <a href='http://code.google.com/apis/searchappliance/kb/secure/kerberos-troubleshooting.html#step8'>Troubleshooting Guide</a>).


## Details ##
System Requirements:
  * 32-bit Windows XP, Vista, Windows 7
  * MIT Kerberos Client (http://web.mit.edu/Kerberos/dist/index.html) installed at (c:\program files\  such that  klist exists at: c:\program files\mit\kerberos\bin\klist.exe)

## Download ##
source:
http://code.google.com/p/gsa-admin-toolkit/source/browse/trunk/gsa_win_utility.hta

download:
http://gsa-admin-toolkit.googlecode.com/svn/trunk/gsa_win_utility.hta

## Usage ##
  * Download the gsa\_win\_toolkit.hta and save it to the desktop with the extension .hta.  The users account running the script does not have to be an admin.
  * Right click on the file, select 'Properties'.  If the security setting blocks execution, select "Unblock".  Depending on your desktop system security, you may not have to do this step.
  * On launch, enter the following
    1. Fully qualified DNS A-Name of the GSA (eg gsa.yourdomain.com)
    1. User account for the GSA in active directory associated with the keytab.  (omit the domain information. eg: just gsauser not DOMAIN\gsauser or gsauser@DOMAIN)
    1. Select the latest keytab
  * The utility will run through about 20+ individual tests comparing the keytab/DNS/AD entry, etc.  Output in green is a pass, in red is an error that the tool detected in the confiuration.

## Error Messages ##
  * On Launch, if you see "Safety settings on this computer prohibit a data source on another domain"
> > Open Internet Explorer <br />
> > Go to Tools -> Internet Options<br />
> > Select the [Security](Security.md) tab <br />
> > Select [Internet](Internet.md) from the list of web content zone <br />
> > Click [Level](Custom.md) <br />
> > Enable [data sources across domains](Access.md) <br />
> > Click all [OK](OK.md) buttons to save and close all settings related pages <br />
> > Try running the diagnostic tool again <br />


## References ##

http://code.google.com/apis/searchappliance/documentation/60/secure_search/secure_search_crwlsrv.html#kerberos_keytab

http://web.mit.edu/Kerberos/dist/index.html