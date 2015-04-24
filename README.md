Migration Script: OpenBSD Local Users to YP LDAP ypldap
=======================================================


I wrote this script to pull in users from files in `/etc` into `ldif` files
 for use with OpenBSD's `ldapd` and `ypldap`.


This Awk script will read users and groups from
 `/etc/master.passwd` and `/etc/group`. By default it will only pull
 users and groups with a `UID` or `GID` in the range `1000` to `3000`.
 At the top of the script you can configure the `UID` and `GID` range which will be copied.


The following `ldif` files will be created:


* base.ldif = Contains the structure of the LDAP database
* group.ldif = Contains the groups read from `/etc/group`
* passwd.ldif = Contains the users read in from `/etc/master.passwd`


---


**/ldap**


Included in the `ldap` directory is a set of LDAP `schema` files for use with
 OpenBSD's `ldapd` and `ypldap`.


 I have modified the `nis.schema` which ships with OpenBSD-5.7 in order
 for the `posixAccount` to support the attributes
 `shadowPassword`, `shadowLastChange`, `shadowExpire`, and `userClass`.
 You must use these `ldap schema` files in order to use this `ypldap` system.


* shadowPassword = Stores the users OpenBSD Blowfish password hash used by YP
    - userPassword = set to `{BSDAUTH}username` so LDAP Binds happen ageist YP and the Blowfish password hash
* shadowLastChange = Used as time by which user must change their password: `change`
* shadowExpire = Use as time the user's account expired: `expire`
* userClass = Used as the users Login Class: `class`


---


**/etc**


Included in the `etc` directory are configuration files for OpenBSD's `ldapd` and `ypldap`


With OpenBSD configured in this way, user authentication happens completely natively.
 Users can authenticate agents the `default` login class. The passwords are
 checked against a normal OpenBSD Blowfish password hash via the YP wrapper service `ypldap`.
 LDAP Binds are checked via {BSDAUTH} which uses YP to check the Blowfish password hash.



### Tested and Used on:


- [OpenBSD 5.7](http://www.openbsd.org/ "OpenBSD 5.7")
- [OpenBSD 5.7 ldapd](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ldapd.8?query=ldapd "OpenBSD 5.7 ldapd")
- [OpenBSD 5.7 ypldap](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man8/ypldap.8?query=ypldap "OpenBSD 5.7 ypldap")


### Using the migration script


You must have `openldap-client` installed.


        pkg_add openldap-client


If you have the file `/etc/myname` on your system with your domain name in it,
 you can simply run the script with no parameters. If you do not have that file
 or if you simply want to create the `ldif` file for a different domain name,
 you can simply specify the domain name as the first parameter.


The script must be run as the `root` user, so `/etc/master.passwd` can be read.


        ypldap-migration
        #  or specify the domain
        ypldap-migration example.com


Then you can add the `ldif` files to your running OpenBSD `ldapd` server.


        ldapadd -H ldap://ldap01.example.com -D 'cn=admin,dc=example,dc=com' -x -W -f base.ldif
        ldapadd -H ldap://ldap01.example.com -D 'cn=admin,dc=example,dc=com' -x -W -f group.ldif
        ldapadd -H ldap://ldap01.example.com -D 'cn=admin,dc=example,dc=com' -x -W -f passwd.ldif


You will want to `compact` and `index` your OpenBSD `ldapd` database.


        ldapctl compact
        ldapctl index


####### vim: set ts=4 sw=4 tw=80 et :######
