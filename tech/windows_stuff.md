# Windows Stuff
I work pretty rarely with Windows, but it's an unusual creature and poorly documented, so I figure I'll put some of my hard-won understandings about AD in my personal notes so that I don't lose them.

# Samba & File Explorer
WSLH has their Linux environment pretty tightly integrated into Active Directory. This seemed strange to me since AD regularly changes as a protocol and so Linux support is flakey and inconsistent. Samba doesn't respond well to connectivity loss, is generally recognized as insecure, and has a very different access model than Linux.

One of the big reasons for the effort put into this tight integration is that Windows File Explorer will allow you to reference a shae with the pattern \\server\share\folder . This pattern means that most files can be offered via Samba, and used by teams across the lab without having to teach users about command lines or learn how to navigate Linux. Because of this I'll have to be very careful of any plans to decouple our Linux hosts from AD and find similarly convenient ways for people to consume subsets of the files. It's possible Windows likes NFS enough that I can do something similar.

# Active Directory CNAME failover
Kerberos is the primary authentication protocol behind Active Directory. The Windows AD server mostly acts as a kind of database ontop of the Kerberos install. When hosts are added to AD, they authethenticate into Kerberos and create entries in this database with their identifying information. So when the hostname changes (e.g. a cname is moved from one host to another), you need to delete these entries so that a server that authenticates can create new entries in the database.

If you are unsure whether AD is going to authenticate with your CNAME, run `sudo klist -ke` on your host. The entries starting with `hosst/` are the names that this server can use to authenticate with Kerberos.

In Active Directory Users and Computers (ADUC) on Windows:
* Update the DNS record so that it points to the new desired primary
* If the old primary is being powered down, use `realm leave`to un-join from AD
* If the old primary is remaining up, remove the CNAME from ADUC, `Computers` or `Domain Member Servers`
* When the CNAME has been replicated to all participating Domain Controllers, re-run a `kinit` on the host. 
* You may need to leave and re-enter the domain?
* In order to verify that this host can now authenticate, run `klist -ke` on the host and verify that a new `host/` entry appears with the CNAME

# Force Active Directory Replication
AD Replication happens relatively infrequently, so it can help to know how to manually kick it off:
* In Active Directory Users and Computers (ADUC), note down which domain controller you operated against
* In Active Directory Sites and Services, navigate to Sites, and select the folder that corresponds to the AD server that you updated.
* For each server in the Servers folder:
    * Navigate to NTDS settings. Select all of the `IP-*` connections, right click, and select Replicate Now
