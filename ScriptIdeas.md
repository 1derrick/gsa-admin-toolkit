# Script Ideas #

Here are some ideas for scripts that would be useful for GSA administrators. If anyone has already written one of these scripts, please let me know.

**Tool to validate XML responses from Authn/Authz SPI**

The appliance does not provide good error messages when using the Authn/Authz SPI. For example, see Google [bug #240337](https://code.google.com/p/gsa-admin-toolkit/issues/detail?id=240337). We can provide appliance admins with a tool that sends a request to their Authn/Authz servers and validates the responses using xmllint.

**Example code for doing field-based search using the appliance**

You can use meta tags to do field-based search on the appliance. For example, show all database records of people who play soccer and live in London. It would be good for us to provide some sample code that GSA administrators can easily modify.

**Configuration changes**

Export the config file and compare to a previous version to determine what config changes have been made to an appliance.

**Testing environment**

RPM or tarball to install and startup all the scripts to configure a testing environment. Also, automatically configure an appliance to use this testing environment.

**Analyze exports from Crawl Diagnostics**

How many URLs crawled each minute etc.