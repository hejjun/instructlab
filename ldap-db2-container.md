# Using LDAP for Db2 Authentication in a Docker Container
Author: EMBER CROOKS
Date: JULY 14, 2020

Db2 is a bit unusual among RDBMSes in that it does not perform authentication. No matter what, you need some external authority to perform authentication. Usually that is either the OS or LDAP, though there are other options. If using LDAP, either transparent LDAP or security plugins can be used.

One of the main reasons I prefer transparent ldap over security plugins is that transparent LDAP allows you to have a fall-back of local ids to login in the case that your LDAP server is unreachable. While Db2 is introducing some caching that may be applicable to security plugins, without it you could find yourself unable to interact with your database when LDAP is down.

I’ve been doing a fair amount of work on containers lately and I ran into some challenges getting LDAP authentication working in a container. I work with an awesome engineer who happens to have his own excellent blog, The Rubyist Blog. While Jon writes a lot about Ruby and is a true wizard with Ruby, he’s also really good with a wide range of technologies. If you’re looking to try working with LDAP in a Db2 container, start with his blog on LDAP in Containers. That will give you a basis for the rest of the content in this article.

Using LDAP for authentication in Db2 containers is a requirement where I work. We have a number of environments running in a Docker/Kubernetes environment that we need to have decent access control for, and we have no desire to try to manage users at the container level. We also have a nice mature LDAP implementation with role-based groups for database access already defined.

## Security Plugins
Honestly, using a security plugin for LDAP is a pretty good idea. The part that always gets to me is that if LDAP were down, I couldn’t even do ANYTHING with my Db2 database. I get the argument that I’ve heard on the other side of this – that if LDAP is down, you’ve got a bigger problem anyway. But something in me just won’t let me give up the ability to get to the database in the most possible situations. If you’re looking for details on how to make security plugins for LDAP work in a container, I haven’t seen a recent decent article on it, but if you have access to IDUG past conference presentations (through having attended the conference, 9 months after the conference through a premium membership, or 18 months after the conference), Ian Bjorhovde included a “recipe” with all the details in his presentation: Docker & Db2: Recipes for Uncontained Success at IDUG North America in 2019.

## Transparent LDAP
After I’d decided to go the transparent LDAP route, there were a number of challenges to solve. Not the least of which was understanding docker and how to build containers. One of the best things to get your head around is NOT to treat a docker container just like another VM or another server. If you do that, you’ll be repeatedly frustrated, particularly if the container is part of a well-built devops infrastructure and is regularly deployed all over again.

I started with this blog on LDAP in Containers.

In running through the process of setup, we discovered, oddly, that somehow just one value in the critical sssd.conf file was being reset, causing ldap to fail entirely when we tried to use it in a newly deployed container. We ended up having to copy our sssd.conf file a second time in our entrypoint script in order to make sure that value didn’t change at the end. In addition to the steps that Jon describes in his blog, we defined a secret for our LDAP password along with several related variables.

The variables we defined included:

* LDAP_ENABLED – this is an indicator, as we have some images we want to use LDAP for and some we don’t (like developer locals).
* LDAP_URI – the uri to our ldap server. This seems fairly static, but we can see future situations where we might want something different.
* LDAP_DEFAULT_BIND_DN – the id that should be used to log in to LDAP.
* LDAP_DEFAULT_AUTHTOK – the password for the id – this is defined as a secret in our helm chart, so we’re never storing this in plain text.

The remainder of things that might have to be defined – like search bases and the certificate location – are all static enough that we just put them in the sssd.conf file that we copy out to the container. In Jon’s blog post, he shows an “sssd.conf” file which is great for testing but it includes a few lines that we need to be dynamic. We removed those lines (lines 13,14, and 16 in his example) plus the last two “nss” lines in our template and add them to the configuration on startup. This allows the same image to be used across environments. You can see how I re-add those lines in the “Build LDAP sssd.conf” section in the script below.

One of the early things I learned was that in the container, I couldn’t use the normal ways of starting sssd. Instead of using the normal systemctl commands to start it as a service, just a straight forward sssd was the way to go.

Another thing we learned was that using the /etc/pam.d/db2 file as provided in the IBM Db2 Knowledge Center was just not necessary. A symbolic link pointing /etc/pam.d/db2 to /etc/pam.d/password-auth worked better that trying to create our own more static file. This is a nugget of information that likely applies to non-containerized transparent LDAP implementations, too.

The portion of our entrypoint script that handles this work is below. Note that we use this both as an entrypoint script for docker containers and also as a script when we build a new EC2 instance using a cloud formation template, so you may not have to have the logic around whether this is a docker container or not.
```
## Add ldap configuration and start ldap
echo ""
echo "Phase 12: Configure LDAP"
echo ${LDAP_ENABLED}
if [[ $LDAP_ENABLED == true ]]; then
  echo "   LDAP is enabled, performing configuration."
  # LDAP parameters
  if [ -z $LDAP_URI ]; then
    LDAP_URI='example.com:1636'
  fi
  if [ -z "$LDAP_DEFAULT_BIND_DN" ]; then
    LDAP_DEFAULT_BIND_DN='uid=someone,ou=something,dc=example,dc=com'
  fi
  if [ -z $LDAP_DEFAULT_AUTHTOK ]; then
    echo "     ERROR: no LDAP password supplied. LDAP will not work until it is changed in sssd.conf"
    exit 4
  fi
  # Copy certificate
  redacted - our methodology for copying and accepting the certificate
  # Build LDAP sssd.conf
  echo "ldap_uri = ldaps://${LDAP_URI}" >> /tmp/sssd.conf
  echo "ldap_default_bind_dn = $LDAP_DEFAULT_BIND_DN" >> /tmp/sssd.conf
  echo "ldap_default_authtok = $LDAP_DEFAULT_AUTHTOK" >> /tmp/sssd.conf
  echo "[nss]" >> /tmp/sssd.conf
  echo "filter_users = root, ldap, named, avahi, haldaemon, dbus, radiusd, news, nscd" >> /tmp/sssd.conf
  # Copy sssd.conf to proper location
  echo "     copying sssd.conf..."
  cp /tmp/sssd.conf /etc/sssd/sssd.conf
  echo "     changing permissions on sssd.conf..."
  chmod 600 /etc/sssd/sssd.conf

  # enable PAM
  authconfig --enablesssdauth --update
  # Copy db2 pam file to right place
  echo "     linking pam db2 file..."
  ln -s /etc/pam.d/password-auth /etc/pam.d/db2
  echo "     starting sssd..."

  # Copying sssd.conf _again_ because something breaks it...
  echo "     copying sssd.conf again..."
  cp /tmp/sssd.conf /etc/sssd/sssd.conf
  if [ -f /.dockerenv ]; then
    #this is docker, start sssd manually
    echo "       docker detected, starting manually..."
    sssd
  else
    # this is not docker, start sssd using systemd
    echo "       not docker, starting via systemd..."
    systemctl restart sssd.service
  fi

  # Clean up temporary sssd config
  echo "     removing copy of sssd.conf from /tmp..."
  rm -f /tmp/sssd.conf

  #set Db2 var for transparent ldap
  echo "    setting DB2AUTH for transparent ldap..."
  su - db2inst1 -c "db2set DB2AUTH=OSAUTHDB"
else
  echo "   LDAP not enabled, skipping phase 12."
fi
```
We also had to add a few things we were installing in the dockerfile itself – We added these to a yum install command that we use to gather all the other package required for the container:

      sssd \
      authconfig \


## Troubleshooting Tips
The work presented in this article occurred over the course of more than a month, sprinkled in with other projects. It seemed like every little issue we worked through, another one popped up. One thing I learned of that I had never heard of before was a little utility that IBM offers to help troubleshoot LDAP. It pointed us straight to pam when something wasn’t working, and was very helpful to pinpoint one of the many problems.

What I found in one iteration was that os-level authentication like su’ing to an id or using getent were working perfectly fine, but every attempt to connect to the database was met with:
```
SQL30082N Security processing failed with reason "24" ("USERNAME AND/OR
PASSWORD INVALID"). SQLSTATE=08001
```
I even had a teammate try to connect as well, since this is one error message that is nearly always caused by user error – the user in this case being me. He confirmed that it wasn’t just me.

When I looked in the db2diagnostic log for more details, I found these entries:
```
2020-06-29-21.06.17.282240+000 I9939854E464          LEVEL: Warning
PID     : 4915                 TID : 139890817754880 PROC : db2sysc 0
INSTANCE: db2inst1             NODE : 000            DB   : SAMPLE
APPHDL  : 0-50
HOSTNAME: db-0
EDUID   : 23                   EDUNAME: db2agent (SAMPLE) 0
FUNCTION: DB2 UDB, bsu security, sqlexLogPluginMessage, probe:20
DATA #1 : String with size, 65 bytes
Password validation for user ecrooks failed with rc = -2146500507
```
Now this error message is not that uncommon in databases where users type in passwords (as opposed to only applications connecting). My brain is wired to ignore it as “user fat-fingered password”, unless I see it for an ID that should not be possible for.

Trying to get more information on this error, I asked Db2 to parse it for me, and it wasn’t all that helpful:
```
db2diag -rc -2146500507

Input ZRC string '-2146500507' parsed as 0x800F0065 (-2146500507).

ZRC value to map: 0x800F0065 (-2146500507)
    V7 Equivalent ZRC value: 0xFFFF8665 (-31131)

ZRC class :
    SQL Error, User Error,... (Class Index: 0)
Component:
    SQLO ; oper system services (Component Index: 15)
Reason Code:
    101 (0x0065)

Identifer:
    SQLO_BAD_PSW
Identifer (without component):
    SQLZ_RC_BADPSW

Description:
    The password is not valid for the specified userid

Associated information:
    Sqlcode -30082
SQL30082N  Security processing failed with reason "" ("").

    Number of sqlca tokens : 2
    Diaglog message number: 8111
```

After a significant amount of web searching, I stumbled across the linuxTransLDAP tool. It is a simple executable for several different platforms, offered by IBM. You just have to transfer it to a system, and you can execute it. I was using the Db2 instance owner to run it. In my case, I executed it using:

`./linuxTransLdap -u ecrooks -p xxxxx`

The user is the ldap user – let’s pretend mine is ecrooks – and the password is specified after the -p flag. As much as I hate specifying a password on the command line, it seems to be necessary here.

That output pointed me at pam specifically and by interacting with Jon, who knows more about LDAP and authentication than I do, we were able to identify that the IBM-supplied values from the IBM Db2 Knowledge center did not directly work for this environment. Instead of independently populating values in /etc/pam.d/db2, we made it a symbolic link to /etc/pam.d/password-auth. This worked like a charm and was the final tweak we needed to fully have transparent LDAP working in a container for Db2.