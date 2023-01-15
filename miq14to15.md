<!-- Output copied to clipboard! -->

<!-----

You have some errors, warnings, or alerts. If you are using reckless mode, turn it off to see inline alerts.
* ERRORs: 0
* WARNINGs: 0
* ALERTS: 2

Conversion time: 2.825 seconds.


Using this Markdown file:

1. Paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs to Markdown version 1.0β34
* Sun Jan 15 2023 04:46:53 GMT-0800 (PST)
* Source doc: miq upgrade 
* Tables are currently converted to HTML tables.
* This document has images: check for >>>>>  gd2md-html alert:  inline image link in generated source and store images to your server. NOTE: Images in exported zip file from Google Docs may not appear in  the same order as they do in your doc. Please check the images!


WARNING:
You have 19 H1 headings. You may want to use the "H1 -> H2" option to demote all headings by one level.

----->


<p style="color: red; font-weight: bold">>>>>>  gd2md-html alert:  ERRORs: 0; WARNINGs: 1; ALERTS: 2.</p>
<ul style="color: red; font-weight: bold"><li>See top comment block for details on ERRORs and WARNINGs. <li>In the converted Markdown or HTML, search for inline alerts that start with >>>>>  gd2md-html alert:  for specific instances that need correction.</ul>

<p style="color: red; font-weight: bold">Links to alert messages:</p><a href="#gdcalert1">alert1</a>
<a href="#gdcalert2">alert2</a>

<p style="color: red; font-weight: bold">>>>>> PLEASE check and correct alert issues and delete this message and the inline alerts.<hr></p>


                                                            

                                

      ManageIQ VMware Appliance

Unofficial Upgrade howto

     Author: [ManageIQ community](https://github.com/ManageIQ/manageiq/discussions) 


# 


# Table of Content


[TOC]





# **Overview** {#overview}

A step by step walkthrough on upgrading miq 14+ appliance to later version.


# 


# **[Glossary](http://manageiq.org/docs/get-started/concepts)** {#glossary}


<table>
  <tr>
   <td>Name
   </td>
   <td>Definition
   </td>
  </tr>
  <tr>
   <td><a href="https://bundler.io/">bundle</a>
   </td>
   <td>Bundler provides a consistent environment for Ruby projects by tracking and installing the exact gems and versions that are needed.
   </td>
  </tr>
  <tr>
   <td>distributed-ruby
   </td>
   <td><a href="http://nithinbekal.com/posts/distributed-ruby/">http://nithinbekal.com/posts/distributed-ruby/</a>
   </td>
  </tr>
  <tr>
   <td>EMS
   </td>
   <td>External management system
   </td>
  </tr>
  <tr>
   <td>vim
   </td>
   <td>Virtual Infrastructure Management
   </td>
  </tr>
  <tr>
   <td><a href="https://bower.io/">bower</a>
   </td>
   <td>A package manager for the web
   </td>
  </tr>
  <tr>
   <td>npm
   </td>
   <td>NPM is a package manager for Node.js packages, or modules if you like.
   </td>
  </tr>
  <tr>
   <td>MIQ all-in-one
   </td>
   <td>miq worker and db on the same machine and region.
   </td>
  </tr>
</table>



## 


# **Miq14t01 before miq 15 upgrade** {#miq14t01-before-miq-15-upgrade}



1. Download miq14 and have it configured with DB role.
2. Adding software (optional)

    ```
    dnf install -y htop nmap molocate emacs-nox xterm && updatedb
    ```


3. Current miq version

    ```
    [root@miq14t01 vmdb]# vmdb && cat VERSION && echo;date
    najdorf-1.3
    Sat Jan 14 08:51:26 CST 2023
    [root@miq14t01 vmdb]#
    ```


4. NTP verification

    ```
    [root@miq14t01 vmdb]# date
    Wed Jan 11 11:22:11 CST 2023
    [root@miq14t01 vmdb]# timedatectl
                   Local time: Wed 2023-01-11 11:22:30 CST
               Universal time: Wed 2023-01-11 17:22:30 UTC
                     RTC time: Wed 2023-01-11 17:22:30
                    Time zone: America/Chicago (CST, -0600)
    System clock synchronized: yes
                  NTP service: active
              RTC in local TZ: no
    [root@miq14t01 vmdb]#
    ```


5.  lv_pg is  hosting local postgresql

    ```
    [root@miq14t01 vmdb]# lvs |egrep 'LV|lv_pg'
      LV               VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      lv_pg            vg_data   -wi-ao---- <13.00g
    [root@miq14t01 vmdb]#
    [root@miq14t01 vmdb]# grep lv_pg /etc/fstab
    /dev/mapper/vg_data-lv_pg /var/lib/pgsql          xfs     defaults        0 0
    [root@miq14t01 vmdb]#
    [root@miq14t01 vmdb]# tree -L 1 /var/lib/pgsql/;date;ls -l /var/lib/pgsql/initdb_postgresql.log;tail -n 5 /var/lib/pgsql/initdb_postgresql.log
    /var/lib/pgsql/
    ├── backups
    ├── data
    └── initdb_postgresql.log

    2 directories, 1 file
    Sat Jan 14 08:55:32 CST 2023
    -rw-------. 1 postgres postgres 907 Dec 24 06:45 /var/lib/pgsql/initdb_postgresql.log

    Success. You can now start the database server using:

        /usr/bin/pg_ctl -D /var/lib/pgsql/data -l logfile start

    [root@miq14t01 vmdb]#
    ```


6. Disk allocation for postgresql /var/lib/postgresql

    ```
    [root@miq14t01 ~]# lvs && pvs
      LV               VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      lv_pg            vg_data   -wi-ao---- <13.00g
      lv_home          vg_system -wi-ao----   1.00g
      lv_os            vg_system -wi-ao----  10.50g
      lv_swap          vg_system -wi-ao----  <6.00g
      lv_tmp           vg_system -wi-ao----   1.00g
      lv_var           vg_system -wi-ao----  12.00g
      lv_var_log       vg_system -wi-ao----  11.00g
      lv_var_log_audit vg_system -wi-ao---- 512.00m
      PV         VG        Fmt  Attr PSize   PFree
      /dev/sda2  vg_system lvm2 a--  <42.00g    0
      /dev/sda5  vg_data   lvm2 a--  <13.00g    0
    [root@miq14t01 ~]#
    ```


7. About this miq rail(bundle exec rake about)
* `Rail info`

    ```
    [root@miq14t01 ~]# vmdb;bundle exec rake about;date
    About your application's environment
    Rails version             6.0.5.1
    Ruby version              ruby 2.7.6p219 (2022-04-12 revision c9c2245c0a) [x86_64-linux]
    RubyGems version          3.1.6
    Rack version              2.2.4
    JavaScript Runtime        Node.js (V8)
    Middleware                SecureHeaders::Middleware, ActionDispatch::HostAuthorization, Rack::Sendfile, ActionDispatch::Executor, ActiveSupport::Cache::Strategy::LocalCache::Middleware, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, ActionDispatch::RemoteIp, Rails::Rack::Logger, ActionDispatch::ShowExceptions, ActionDispatch::DebugExceptions, ActionDispatch::ActionableExceptions, ActionDispatch::Callbacks, ActionDispatch::Cookies, ActionDispatch::Session::MemCacheStore, ActionDispatch::Flash, ActionDispatch::ContentSecurityPolicy::Middleware, Rack::Head, Rack::ConditionalGet, Rack::ETag, Rack::TempfileReaper, RequestStartedOnMiddleware
    Application root          /var/www/miq/vmdb
    Environment               production
    Database adapter          postgresql
    Database schema version   20220114155819
    Wed Jan 11 11:24:52 CST 2023
    [root@miq14t01 vmdb]#

    ```


* `Ports opened on this all-in-one MIQ instance.`

    ```
    [root@miq14t01 vmdb]# nmap localhost;date
    Starting Nmap 7.70 ( https://nmap.org ) at 2023-01-14 08:56 CST
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.000014s latency).
    Other addresses for localhost (not scanned): ::1
    rDNS record for 127.0.0.1: miq14t01.test.lan
    Not shown: 989 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    111/tcp  open  rpcbind
    443/tcp  open  https
    3000/tcp open  ppp
    3001/tcp open  nessus
    4000/tcp open  remoteanything
    4001/tcp open  newoak
    5000/tcp open  upnp
    5432/tcp open  postgresql
    9090/tcp open  zeus-admin

    Nmap done: 1 IP address (1 host up) scanned in 1.64 seconds
    Sat Jan 14 08:56:58 CST 2023
    [root@miq14t01 vmdb]#
    ```



    

8. Current EVM running status


```
[root@miq14t01 vmdb]#  vmdb; bin/rake evm:status;date
Checking EVM status...
 Region | Zone    | Server | Status  |  PID | SPID | Workers | Version     | Started    | Heartbeat   | MB Usage | Roles
--------|---------|--------|---------|------|------|---------|-------------|------------|-------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------
      0 | default | EVM*   | started | 1790 | 2285 |      13 | najdorf-1.3 | 2023-01-11 | 14:57:10UTC |      307 | automate:database_operations:database_owner:ems_inventory:ems_operations:event:remote_console:reporting:scheduler:smartstate:user_interface:web_services

* marks a master appliance

 Type          | Status  |  PID | SPID | Queue     | MB Usage
---------------|---------|------|------|-----------|----------
 EventHandler  | started | 2288 | 2612 | ems       | 181/1024
 Generic       | started | 2286 | 2624 | generic   | 267/1024
 Generic       | started | 2287 | 2619 | generic   | 264/1024
 Priority      | started | 2289 | 2627 | generic   | 213/1024
 Priority      | started | 2290 | 2617 | generic   | 244/1024
 RemoteConsole | started | 2293 |      | http:5000 | 192/1024
 Reporting     | started | 2291 | 2615 | reporting | 181/1024
 Reporting     | started | 2292 | 2631 | reporting | 181/1024
 Schedule      | started | 2299 | 2625 |           | 281/1024
 Ui            | started | 2301 |      | http:3000 | 296/1024
 Ui            | started | 2305 |      | http:3001 | 281/1024
 WebService    | started | 2322 |      | http:4000 | 288/1024
 WebService    | started | 2328 |      | http:4001 | 289/1024

All rows have the values: Region=0, Zone=default, Server=EVM, Started=2023-01-11, Heartbeat=2023-01-11
Sat Jan 14 08:57:35 CST 2023
[root@miq14t01 vmdb]#

```



9. Use command line to stop

    ```
    [root@miq14t01 vmdb]#  bin/rake evm:stop
    Stopping EVM gracefully...
    [root@miq14t01 vmdb]# bin/rake evm:status
    Checking EVM status...
     Region | Zone    | Server | Status  |  PID | SPID | Workers | Version     | Started    | Heartbeat   | MB Usage | Roles
    --------|---------|--------|---------|------|------|---------|-------------|------------|-------------|----------|----------------
          0 | default | EVM    | stopped | 1790 | 2285 |       0 | najdorf-1.3 | 2023-01-11 | 14:58:37UTC |      307 | database_owner

    * marks a master appliance

    [root@miq14t01 vmdb]#
    ```


    1. tbc


## 




# [Miq14 to miq15 migration](https://github.com/ManageIQ/manageiq/discussions/22295) {#miq14-to-miq15-migration}

Procedures provided by[ https://github.com/jrafanie](https://github.com/jrafanie)



1. [Review existing miq 14 environment](#bookmark=id.m38o24j675ya)
2. [Read the miq 15 blog](https://www.manageiq.org/blog/2023/01/manageiq-oparin-ga-announcement/)
3. Take a vmware snapshot if this is a vmware appliance.

    You'll need to do a database backup before you upgrade as postgresql 13 is the new required database version. You can also do `pg_upgrade` if you're familiar with this but it's not covered here.

4. Make sure you have enough space to backup your database to a file.

    ```
    [root@miq14t01 vmdb]# df -lh;lsblk;date
    Filesystem                              Size  Used Avail Use% Mounted on
    devtmpfs                                7.8G     0  7.8G   0% /dev
    tmpfs                                   7.8G  124K  7.8G   1% /dev/shm
    tmpfs                                   7.8G  880K  7.8G   1% /run
    tmpfs                                   7.8G     0  7.8G   0% /sys/fs/cgroup
    /dev/mapper/vg_system-lv_os              11G  4.1G  6.5G  39% /
    /dev/mapper/vg_system-lv_var             12G  1.1G   11G   9% /var
    /dev/mapper/vg_system-lv_home          1014M   40M  975M   4% /home
    /dev/sda3                                10G  104M  9.9G   2% /var/www/miq_tmp
    /dev/mapper/vg_system-lv_tmp           1014M   40M  975M   4% /tmp
    /dev/mapper/vg_system-lv_var_log         11G  486M   11G   5% /var/log
    /dev/mapper/vg_system-lv_var_log_audit  507M   31M  477M   7% /var/log/audit
    /dev/mapper/vg_data-lv_pg                13G  216M   13G   2% /var/lib/pgsql
    /dev/sda1                              1014M  386M  629M  38% /boot
    tmpfs                                   1.6G     0  1.6G   0% /run/user/0
    NAME                           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda                              8:0    0   66G  0 disk
    ├─sda1                           8:1    0    1G  0 part /boot
    ├─sda2                           8:2    0   42G  0 part
    │ ├─vg_system-lv_os            253:0    0 10.5G  0 lvm  /
    │ ├─vg_system-lv_swap          253:1    0    6G  0 lvm  [SWAP]
    │ ├─vg_system-lv_home          253:2    0    1G  0 lvm  /home
    │ ├─vg_system-lv_tmp           253:3    0    1G  0 lvm  /tmp
    │ ├─vg_system-lv_var_log_audit 253:4    0  512M  0 lvm  /var/log/audit
    │ ├─vg_system-lv_var_log       253:5    0   11G  0 lvm  /var/log
    │ └─vg_system-lv_var           253:6    0   12G  0 lvm  /var
    ├─sda3                           8:3    0   10G  0 part /var/www/miq_tmp
    ├─sda4                           8:4    0    1K  0 part
    └─sda5                           8:5    0   13G  0 part
      └─vg_data-lv_pg              253:7    0   13G  0 lvm  /var/lib/pgsql
    Sat Jan 14 10:10:25 CST 2023
    [root@miq14t01 vmdb]#
    ```


5. Install miq15 repo

    ```
    rpm -ivh  https://rpm.manageiq.org/release/15-oparin/el8/noarch/manageiq-release-15.0-1.el8.noarch.rpm
    [root@miq14t01 vmdb]# dnf repolist |grep manage
    manageiq-14-najdorf        ManageIQ 14 (Najdorf) Release- x86_64
    manageiq-14-najdorf-noarch ManageIQ 14 (Najdorf) Release- noarch
    manageiq-15-oparin         ManageIQ 15 (Oparin) Release - x86_64
    manageiq-15-oparin-noarch  ManageIQ 15 (Oparin) Release - noarch
    [root@miq14t01 vmdb]#
    ```


6. Do a backup  in a location with enough space:

    ```
    [root@miq14t01 vmdb]# pg_dumpall -c --if-exists | gzip > vmdb.pg.gz; ls -l vmdb.pg.gz
    -rw-r--r--. 1 root root 361895 Jan 14 09:04 vmdb.pg.gz
    [root@miq14t01 vmdb]#
    ```


7. Clear data 

    You'll need to clear the data directory since postgresql 10 is not compatible with postgresql 13. Ensure your appliance is fully backed up and the database backup file is not in the data directory.


    ```
    systemctl stop postgresql
    rm -rf $APPLIANCE_PG_DATA
    [root@miq14t01 vmdb]# ls -l /var/lib/pgsql/data
    ls: cannot access '/var/lib/pgsql/data': No such file or directory
    [root@miq14t01 vmdb]# ls -l /var/lib/pgsql/
    total 4
    drwx------. 2 postgres postgres   6 Dec 17  2021 backups
    -rw-------. 1 postgres postgres 907 Dec 24 06:45 initdb_postgresql.log
    [root@miq14t01 vmdb]#
    ```


8. Enable module repo for postgresql and ruby

    After modifying the dnf repo information, backing up your database, and clearing the data directory:


    ```
    dnf -y module reset postgresql && dnf -y module enable postgresql:13
    dnf -y module reset ruby && dnf -y module enable ruby:3.0

    [root@miq14t01 vmdb]# yum repolist
    repo id                                                                repo name
    ansible-runner                                                         Ansible Runner for EL 8 - x86_64
    appstream                                                              CentOS Stream 8 - AppStream
    baseos                                                                 CentOS Stream 8 - BaseOS
    epel                                                                   Extra Packages for Enterprise Linux 8 - x86_64
    epel-next                                                              Extra Packages for Enterprise Linux 8 - Next - x86_64
    extras                                                                 CentOS Stream 8 - Extras
    extras-common                                                          CentOS Stream 8 - Extras common packages
    manageiq-14-najdorf                                                    ManageIQ 14 (Najdorf) Release- x86_64
    manageiq-14-najdorf-noarch                                             ManageIQ 14 (Najdorf) Release- noarch
    manageiq-15-oparin                                                     ManageIQ 15 (Oparin) Release - x86_64
    manageiq-15-oparin-noarch                                              ManageIQ 15 (Oparin) Release - noarch
    [root@miq14t01 vmdb]#
    ```


9. Upgrade to ruby 3.0 and postgresql 13 and remove old ones
    1. upgrade

        ```
        dnf -y upgrade --allowerasing
        ```


    2. Preview what will be removed

    ```
    [root@miq14t01 vmdb]# dnf  autoremove
    Last metadata expiration check: 1:26:25 ago on Sat 14 Jan 2023 07:53:44 AM CST.
    Dependencies resolved.
    ========================================================================================================================================================================
     Package                                Architecture           Version                                                            Repository                       Size
    ========================================================================================================================================================================
    Removing:
     ansible-runner                         noarch                 1.4.7-1.el8                                                        @ansible-runner                   0
     grub2-tools-efi                        x86_64                 1:2.02-129.el8                                                     @baseos                         2.0 M
     nodejs                                 x86_64                 1:14.21.1-2.module_el8.7.0+1235+834c02a1                           @appstream                       37 M
     nodejs-docs                            noarch                 1:14.21.1-2.module_el8.7.0+1235+834c02a1                           @appstream                       65 M
     nodejs-full-i18n                       x86_64                 1:14.21.1-2.module_el8.7.0+1235+834c02a1                           @appstream                       28 M
     npm                                    x86_64                 1:6.14.17-1.14.21.1.2.module_el8.7.0+1235+834c02a1                 @appstream                       15 M
     python3-ansible-runner                 noarch                 1.4.7-1.el8                                                        @ansible-runner                 341 k
     python3-daemon                         noarch                 2.2.4-1.el8                                                        @epel                           115 k
     python3-docutils                       noarch                 0.14-12.module_el8.5.0+761+faacb0fb                                @appstream                      5.9 M
     python3-lockfile                       noarch                 1:0.11.0-13.el8.1                                                  @epel                            81 k
     samba                                  x86_64                 4.17.2-2.el8                                                       @baseos                         5.8 M

    Transaction Summary
    ========================================================================================================================================================================
    Remove  11 Packages

    Freed space: 159 M
    Is this ok [y/N]:
    ```


    3. Now do the 

        ```
        dnf -y autoremove
        ```


    4. end
10. Version check

    Check `ruby -v` and `psql --version` show 3.0 and 13.x.


    ```
    [root@miq14t01 vmdb]# ruby -v && psql --version && date
    ruby 3.0.4p208 (2022-04-12 revision 3fa771dded) [x86_64-linux]
    psql (PostgreSQL) 13.7
    Sat Jan 14 09:21:17 CST 2023
    [root@miq14t01 vmdb]#
    ```


11. Initialize a new data directory with pg 13:

    ```
    appliance_console_cli --internal --standalone --password smartvm
    [root@miq14t01 vmdb]# appliance_console_cli --internal --standalone --password smartvm
    configuring internal database
    Initialize postgresql starting
    Initialize postgresql complete
    [root@miq14t01 vmdb]#
    ```


12. Restore the db with postgresql 13

    ```
    gunzip <  <your_backup_file>  | psql postgres
    [root@miq14t01 vmdb]# gunzip < ./vmdb.pg.gz  | psql postgres
    <snipped>
    CREATE INDEX
    CREATE INDEX
    CREATE INDEX
    CREATE INDEX
    CREATE INDEX
    CREATE INDEX
    CREATE TRIGGER
    CREATE TRIGGER
    [root@miq14t01 vmdb]# gunzip < ./vmdb.pg.gz  | psql postgres
    ```


13. Do the db migrate

    Run `rake db:migrate` and `rake evm:automate:reset`. 


    ```
    [root@miq14t01 vmdb]# rake db:migrate && rake evm:automate:reset ;date
    == 20220202122417 AddParentRelationToMiqRequests: migrating ===================
    -- add_reference(:miq_requests, :parent, {:type=>:bigint, :_uses_legacy_reference_index_name=>true})
       -> 0.0047s
    == 20220202122417 AddParentRelationToMiqRequests: migrated (0.0048s) ==========

    == 20220223095704 RemoveCol3FromMiqWidgetSet: migrating =======================
    -- Moving col3 widgets to col2 and removing col3
    <snipped>
    == 20221114165219 RemoveVimStringFromMiqRequestTaskOptions: migrating =========
    -- Removing VimStrings from MiqRequestTask
       -> Processing 0 rows
       -> 0.0199s
    == 20221114165219 RemoveVimStringFromMiqRequestTaskOptions: migrated (0.0201s)

    Resetting the default domains in the automation model
    The default domains in the automation model have been reset.
    Sat Jan 14 09:26:45 CST 2023
    [root@miq14t01 vmdb]#
    ```


14. Restart miq

    Then you can restart the appliance or just `systemctl start evmserverd`


    ```
    [root@miq14t01 vmdb]#  systemctl start evmserverd
    [root@miq14t01 vmdb]# systemctl status evmserverd
    ```


15. Post miq15 migration checks

    ```
    systemctl status evmserverd
    [root@miq14t01 vmdb]# vmdb;bundle exec rake about;date
    About your application's environment
    Rails version             6.1.7
    Ruby version              ruby 3.0.4p208 (2022-04-12 revision 3fa771dded) [x86_64-linux]
    RubyGems version          3.2.33
    Rack version              2.2.5
    Middleware                SecureHeaders::Middleware, ActionDispatch::HostAuthorization, Rack::Sendfile, ActionDispatch::Executor, ActiveSupport::Cache::Strategy::LocalCache::Middleware, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, ActionDispatch::RemoteIp, Rails::Rack::Logger, ActionDispatch::ShowExceptions, ActionDispatch::DebugExceptions, ActionDispatch::ActionableExceptions, ActionDispatch::Callbacks, ActionDispatch::Cookies, ActionDispatch::Session::MemCacheStore, ActionDispatch::Flash, ActionDispatch::ContentSecurityPolicy::Middleware, ActionDispatch::PermissionsPolicy::Middleware, Rack::Head, Rack::ConditionalGet, Rack::ETag, Rack::TempfileReaper, RequestStartedOnMiddleware
    Application root          /var/www/miq/vmdb
    Environment               production
    Database adapter          postgresql
    Database schema version   20221114165219
    Sat Jan 14 09:31:31 CST 2023
    [root@miq14t01 vmdb]#
    ```


16. GUI verification by login into miq GUI.

    

<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


17. Delete VMWare snapshot
18. Post comment

This assumes you're upgrading an appliance running both the application and database. If you're running just the application and not the database, you only need to upgrade the packages via the dnf commands. If you're running just a database, you'll need to do the dnf commands and database commands and leave the application steps to the first appliance connecting to this appliance.



19. End


# 


# MIQ data disk size expension {#miq-data-disk-size-expension}

Default data partition is too small to semi-long term test environment. So we need to add 30G 2nd disk from vcenter and extend os partition for 10G using lvextend command.



1. `Add new 30G disk from vcenter.`
    1. `Fdsik /dev/sdc to create /dev/sdc1`
    2. `Pvcreate /dev/sdc1`
2. `Look up existing disk layout`

    ```
    # 
    [root@localhost ~]# pvs
      PV         VG        Fmt  Attr PSize  PFree
      /dev/sda2  vg_system lvm2 a--  26.00g     0
      /dev/sda5  vg_data   lvm2 a--  13.00g     0
      /dev/sdb1            lvm2 ---  20.00g 20.00g
      /dev/sdc1            lvm2 ---  20.00g 20.00g
    [root@localhost ~]# vgextend vg_system /dev/sdb1

    ```


3. `Extend the disk size`

    ```
    # vgextend vg_data /dev/sdc1
    # lvextend -L+10G /dev/mapper/vg_system-lv_os -r

    ```


4. `End`


# 


# VMWare DDK 7.0 install {#vmware-ddk-7-0-install}



1. [VMware Customer Connect](https://customerconnect.vmware.com/dashboard)
2. [Download VMware vSphere Virtual Disk Development Kit 7.0U1](https://customerconnect.vmware.com/downloads/get-download?downloadGroup=VDDK70U1)
3. VMWare 7.0 SDK download

        

<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


    1. MIQ Vmware 7.0 sdk Installation on centos 8.2.

        ```
        # dnf install -y rsync                
        # tar xzvf VMware-vix-disklib-7.0.0-15832853.x86_64.tar.gz
        mkdir /usr/lib/vmware-vix-disklib-7.0
        cd /root/vmware-vix-disklib-distrib
        rsync -azpv bin64 include lib64 /usr/lib/vmware-vix-disklib-7.0
        ln -s /usr/lib/vmware-vix-disklib-7.0/lib64/libvixDiskLib.so /usr/lib/libvixDiskLib.so
ln -s /usr/lib/vmware-vix-disklib-7.0/lib64/libvixDiskLib.so.7 /usr/lib/libvixDiskLib.so.7
reboot
        ldconfig # to load the config
        ldconfig -p | grep -i vix
        [root@miqregion01 ~]# ldconfig -p | grep -i vix
                libvixDiskLib.so (libc6,x86-64) => /lib/libvixDiskLib.so
        [root@miqregion01 ~]#
        ```


    2. 6.7

        ```
        rsync -azpv bin include lib /usr/lib/vmware-vix-disklib/
        ln -s /usr/lib/vmware-vix-disklib/lib/libvixDiskLib.so /usr/lib/libvixDiskLib.so
ln -s /usr/lib/vmware-vix-disklib/lib/libvixDiskLib.so.6 /usr/lib/libvixDiskLib.so.6
        ```


    3. end


# 


# **Errors and solution** {#errors-and-solution}



1. `Tbc`
2. `end`


# 


# **File Listing** {#file-listing}



1. /root/clean_manageiq.sh

    ```
    [root@miqr10db01 ~]# cat /root/clean_manageiq.sh
    #!/bin/bash
    # clean_manageiq.sh
    # Stop and disable services
    # This is from miq team also

    systemctl stop evmserverd   &&  systemctl disable evmserverd
    systemctl stop $APPLIANCE_PG_SERVICE && systemctl disable $APPLIANCE_PG_SERVICE

    # Remove auto-generated files
    if [ -d /var/www/miq/vmdb ]; then
      pushd /var/www/miq/vmdb
        rm -f REGION GUID certs/v2_key certs/server.cer.key certs/server.cer  config/database.yml
      popd
    else
      echo "/var/www/miq/vmdb not found."
      exit 1
    fi

    # Remove pg database data
    if [ -d "$APPLIANCE_PG_DATA" ]; then
      pushd $APPLIANCE_PG_DATA
      rm -rf ./*
      popd
    else
      echo "$APPLIANCE_PG_DATA not found"
    fi

    exit 0
    [root@miqr10db01 ~]#
    ```


2. End


# 


# **Revision History** {#revision-history}


<table>
  <tr>
   <td>Date
   </td>
   <td>who
   </td>
   <td>comment
   </td>
  </tr>
  <tr>
   <td>01/14/2023
   </td>
   <td>tjyang
   </td>
   <td>Init this of doc using material from adam
   </td>
  </tr>
  <tr>
   <td>02/08/2019
   </td>
   <td>caros
   </td>
   <td>Provide pg_dump approach to upgrade across miq releases.
   </td>
  </tr>
</table>



# 


# **References ** {#references}



1. Other gdoc regarding miq, initiated by tjyang.
    1. [MIQ REST API Tutorial MIQ REST API Tutorial ](https://docs.google.com/document/d/1I3eujnF7XHyJQfUQfVUO3oDV_VEtHxXjwZD-sBWK-BA/edi)
    2. [MIQ Multi-Regions Workbook](https://docs.google.com/document/d/1Yty9zDSl9ga7BXNhDVEns4m4beSjUiR_FM-4kWZg_e4/)
2. [http://talk.manageiq.org/t/miq-appliance-upgrade/755](http://talk.manageiq.org/t/miq-appliance-upgrade/755)
3. [https://pemcg.gitbooks.io/mastering-automation-in-cloudforms-4-2-and-manage/content/peng_under_the_hood/chapter.html](https://pemcg.gitbooks.io/mastering-automation-in-cloudforms-4-2-and-manage/content/peeping_under_the_hood/chapter.html)
4. [epihttp://talk.manageiq.org/t/miq-vmware-appliance-upgrade-howto/2397/3](https://pemcg.gitbooks.io/mastering-automation-in-cloudforms-4-2-and-manage/content/peeping_under_the_hood/chapter.html)
5. [http://talk.manageiq.org/t/miq-appliance-upgrade/755/11](http://talk.manageiq.org/t/miq-appliance-upgrade/755/11)
6. End