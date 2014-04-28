- PostgreSQL 9.3.4:

Need to use information related here:

http://www.c0t0d0s0.org/archives/4074-Less-known-Solaris-features-RBAC-and-Privileges-Part-2-Role-based-access-control.html:

# Using authorizations for Services
Although applications need support for this model, you can even use it as an admin. SMF has build in support for authorizations. You can assign authorizations to a service. Every role or user with this authorization is allowed to work with the service (restarting,stop,start,status, ...). Let´s use Apache for this example.

A normal user has no permission to restart the service:

    jmoekamp$ /usr/sbin/svcadm -v disable -s apache2
    svcadm: svc:/network/http:apache2: Couldn't modify "general" property group (permission denied).

Wouldn´t it be nice, to have an authorization that enables a regular user to restart it? Okay, no problem. Let´s create one:

    $ su root
    # echo "solaris.smf.manage.apache/server:::Apache Server management::" >> /etc/security/auth_attr

That´s all. Where is the definition of the permission that the authorisation measn? There is no definition. It´s the job of the application to work with.

Now assign this authorization to the user:

    # usermod -A solaris.smf.manage.apache/server jmoekamp
    UX: usermod: jmoekamp is currently logged in, some changes may not take effect until next login.

Okay, but at the moment no one checks for this authorization, as no application is aware of it. We have to tell SMF to use this authorization.

The authorizations for an SMF servers is part of the general properties of the service. Let´s have a look at the properties of this services.

    # svcprop -p general apache2
    general/enabled boolean false
    general/entity_stability astring Evolving

No authorization configured. Okay ... let´s add the authorization we´ve defined before:

    svccfg -s apache2 setprop general/action_authorization=astring: 'solaris.smf.manage.apache/server'

Check the properties again:

    # svcadm refresh apache2
    # svcprop -p general apache2
    general/enabled boolean false
    general/action_authorization astring solaris.smf.manage.apache/server
    general/entity_stability astring Evolving

Okay, a short test. Exit your root shell and login as the regular user you have assigned the authorization.

    bash-3.2$ svcs apache2
    STATE STIME FMRI
    disabled 22:49:51 svc:/network/http:apache2

Okay, i can view the status of the service. Now i try to start it.

    bash-3.2$ /usr/sbin/svcadm enable apache2
    svcadm: svc:/network/http:apache2: Permission denied.

What the hell ...? No permission to start the service? Yes, enabling the service is not only a method (the start up script), it´s a value of a certain parameter. When you only have the action_authorization you can only do task, that doesn´t change the state of the service. You can restart it (no change of the service properties), but not enable or disable it (a change of the service properties). But this is not a problem. You have to login as root again and assign the the solaris.smf.manage.apache/server authorisation to the value authorisation.

    # svccfg -s apache2 setprop general/value_authorization=astring: 'solaris.smf.manage.apache/server'

With the value authorisation SMF allows you to change the state of the service. Try it again.

    bash-3.2$ /usr/sbin/svcadm enable apache2
    bash-3.2$

  Copy files into, respectively:

  /lib/svc/method/postgres9

  /lib/svc/manifest/application/database/postgres9.xml

  Get things going with:

  svcadm restart system/manifest-import
