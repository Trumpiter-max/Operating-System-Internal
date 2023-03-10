# Chapter 2: Securing User Accounts

Make sure that users can always access their stuff and that they can perform the required tasks to do their jobs

---

## Table of content

- [Chapter 2: Securing User Accounts](#chapter-2-securing-user-accounts)
  - [Table of content](#table-of-content)
  - [The dangers of logging in as the root user](#the-dangers-of-logging-in-as-the-root-user)
  - [The advantages of using sudo](#the-advantages-of-using-sudo)
  - [Setting up sudo privileges for full administrative users](#setting-up-sudo-privileges-for-full-administrative-users)
  - [Adding users to a predefined admin group](#adding-users-to-a-predefined-admin-group)
  - [Creating an entry in the sudo policy file](#creating-an-entry-in-the-sudo-policy-file)
  - [Setting up sudo for users with only certain delegated privileges](#setting-up-sudo-for-users-with-only-certain-delegated-privileges)
  - [Hands-on lab for assigning limited sudo privileges](#hands-on-lab-for-assigning-limited-sudo-privileges)
  - [The sudo timer](#the-sudo-timer)
  - [View your sudo privileges](#view-your-sudo-privileges)
  - [Hands-on lab for disabling the sudo timer](#hands-on-lab-for-disabling-the-sudo-timer)
  - [Preventing users from having root shell access](#preventing-users-from-having-root-shell-access)
  - [Preventing users from using shell escapes](#preventing-users-from-using-shell-escapes)
  - [Preventing users from using other dangerous programs](#preventing-users-from-using-other-dangerous-programs)
  - [Limiting the user's actions with commands](#limiting-the-users-actions-with-commands)
  - [Letting users run as other users](#letting-users-run-as-other-users)
  - [Preventing abuse via user's shell scripts](#preventing-abuse-via-users-shell-scripts)
  - [Detecting and deleting default user account](#detecting-and-deleting-default-user-account)
  - [Locking down users' home directories the Red Hat or CentOS way](#locking-down-users-home-directories-the-red-hat-or-centos-way)
  - [Locking down users' home directories the Debian/Ubuntu way](#locking-down-users-home-directories-the-debianubuntu-way)
  - [useradd on Debian/Ubuntu](#useradd-on-debianubuntu)
  - [adduser on Debian/Ubuntu](#adduser-on-debianubuntu)
  - [Hands-on lab for configuring adduser](#hands-on-lab-for-configuring-adduser)
  - [Enforcing strong password criteria](#enforcing-strong-password-criteria)
  - [Installing and configuring pwquality](#installing-and-configuring-pwquality)
  - [Hands-on lab for setting password complexity criteria](#hands-on-lab-for-setting-password-complexity-criteria)
  - [Setting and enforcing password and account expiration](#setting-and-enforcing-password-and-account-expiration)
  - [Configuring default expiry data for useradd for Red Hat or CentOS only](#configuring-default-expiry-data-for-useradd-for-red-hat-or-centos-only)
  - [Setting expiry data on a per-account basis with useradd and usermod](#setting-expiry-data-on-a-per-account-basis-with-useradd-and-usermod)
  - [Setting expiry data on a per-account basis with chage](#setting-expiry-data-on-a-per-account-basis-with-chage)
  - [Hands-on lab for setting account and password expiry data](#hands-on-lab-for-setting-account-and-password-expiry-data)
  - [Preventing brute-force password attacks](#preventing-brute-force-password-attacks)
  - [Configuring the pam\_tally2 PAM](#configuring-the-pam_tally2-pam)
  - [Hands-on lab for configuring pam\_tally2](#hands-on-lab-for-configuring-pam_tally2)
  - [Locking user accounts](#locking-user-accounts)
  - [Using usermod to lock a user account](#using-usermod-to-lock-a-user-account)
  - [Using passwd to lock user accounts](#using-passwd-to-lock-user-accounts)
  - [Locking the root user account](#locking-the-root-user-account)
  - [Setting up security banners](#setting-up-security-banners)
  - [Using the motd file](#using-the-motd-file)
  - [Using the issue file](#using-the-issue-file)
  - [Using the issue.net file](#using-the-issuenet-file)
  - [Detecting compromised passwords](#detecting-compromised-passwords)
  - [Hands-on lab for detecting compromised passwords](#hands-on-lab-for-detecting-compromised-passwords)
  - [Understanding centralized user management](#understanding-centralized-user-management)
  - [Microsoft Active Directory](#microsoft-active-directory)
  - [Samba on Linux](#samba-on-linux)
  - [FreeIPA/Identity Management on RHEL/CentOS](#freeipaidentity-management-on-rhelcentos)

---

## The dangers of logging in as the root user

The root user can present a whole load of security problems, and can do the following:
 - Accidentally causes damage to the system
 - Someone else causes damage to the system

`sudo` utility allows users to perform administrative tasks without incurring the risk of having them always log on as the root user, and that would also allow users to have only the admin privileges they need to perform a certain job

---

## The advantages of using sudo

 - Assign certain users full administrative privileges
 - Allow users to perform administrative tasks
 - Make it harder for intruders to break into your systems
 - Improve your auditing capabilities
 - Create `sudo` policies to deploy across an entire enterprise network

---

## Setting up sudo privileges for full administrative users

## Adding users to a predefined admin group
 - In a business setting, allowing people to have password-less sudo privileges is a definite no-no
 - Some commands to config:
    - `groups` list user group file
    - `sudo visudo`, I'll open the sudo policy file then go to line `## Allows people in group <group-name>`
    - `usermod` add an existing user to the group with the -G option (Ex: `sudo usermod -a -G <group-name> <user-name>`)
    - `useradd` add a user account to the group as creating it (Ex: `sudo useradd -G <group-name> <user-name>`)
    - `sudo passwd -l root` disable the root account  

## Creating an entry in the sudo policy file

Config with with some command after using `sudo visudo` (in `/etc/sudoers` or simply `sudoer`):
 - `User_Alias ADMINS = <user-name-1>, <user-name-2>` set add own set of usernames, or add a line with own user alias
 - `ADMINS (or <user-name>) ALL=(ALL) ALL` give members of the user alias or special user full sudo power

## Setting up sudo for users with only certain delegated privileges

 - `User_Alias SOFTWAREADMINS = <user-name-1>, <user-name-2>` create other user aliases for other purposes
 - `Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum` is example command aliases
 - `SOFTWAREADMINS ALL=(ALL) SOFTWARE` assign the `SOFTWARE` command alias to the SOFTWAREADMINS user alias
 - `Host_Alias FILESERVERS = fs1, fs2` is host aliases example preceding the user alias

## Hands-on lab for assigning limited sudo privileges

```sh
  #!/bin/bash

  echo -e "Add new users"

  users=(lionel katelyn maggie);

  for user in "${users[@]}";
  do
      if ((`grep -c $user /etc/passwd` > 0));
      then
          echo "Account: $user exists";
      else
          echo -e "\n== Register account $user ==\n"
          sudo useradd $user
          sudo passwd $user
      fi;
  done

  # delete exist user with userdel <user-name>

  if ((`grep -c lionel /etc/sudoers` > 0));
  then    
      echo "lionel exists";
  else
      sudo echo "lionel ALL=(ALL) ALL" >> /etc/sudoers;
  fi;

  if ((`grep -c katelyn /etc/sudoers` > 0));
  then    
      echo "katelyn exists";
  else
      sudo echo "katelyn ALL=(ALL) /usr/bin/systemctl status sshd" >> /etc/sudoers;
  fi;

  if ((`grep -c maggie /etc/sudoers` > 0));
  then    
      echo "maggie exists";
  else
      sudo echo "maggie ALL=(ALL) STORAGE" >> /etc/sudoers;
  fi;

  echo -e "Modify sudoer file done \n"
  shift # shift all parameters

```

---

## The sudo timer

 - By default, the sudo timer is set for five minutes (require password again for `sudo` command per 5 minutes)
 - Changing with `sudo visudo` then find and modify this line `Defaults        timestamp_timeout=XXXX` (replace `XXXX` to exactly minutes). Moreover, disable timeout `timestamp_timeout` to `!authenticate` 
 - Make this a global setting for all users, or set it for certain individual users
 - `sudo -k` reset the sudo timer

## View your sudo privileges

`sudo -l` see some of the environmental variables for user account and privileges

## Hands-on lab for disabling the sudo timer

```sh
  echo "Normal timer for `whoami`"
  sleep 1

  # user just type password once
  sudo fdisk -l
  sudo systemctl status sshd
  sudo iptables -L

  sleep 1
  clear

  echo "Using reset timer"
  sleep 1

  sudo fdisk -l
  sudo -k # reset timer that make user type password again
  sudo fdisk -l

  echo "Force user typer password every time"
  sudo echo "Defaults timestamp_timeout = 0" >> /etc/sudoers

  echo "Modify timestamp_timeout for only user lionel"

  sudo sed 's/Defaults timestamp_timeout = 0/Defaults:lionel timestamp_timeout = 0/' /etc/sudoers

  echo "View current user own sudo privileges"
  sudo -l
```

---

## Preventing users from having root shell access

 - `<user-name> ALL=(ALL) /bin/bash, /bin/zsh` full access for the user so don't add lines like this to sudoers because it will trouble
 - Set up a user with limited sudo privileges

## Preventing users from using shell escapes

 - Certain programs, especially text editors and pagers, have a handy shell escape feature. This allows a user to run a shell command without having to exit the program first (Ex. `:!ls` to someone could run the `ls` command by running in Vi/Vim/emacs/less/view/more)
 - This use Vim's shell escape feature to perform other root-level commands, which includes being able to edit other configuration files
 - Fix this problem by having user use `sudoedit` (no shell escape feature) instead of vim

## Preventing users from using other dangerous programs

 - Give users unrestricted privileges to use dangerous programs (Ex. cat, cut, awk, sed, etc.)

## Limiting the user's actions with commands

 - sudo rule so that user can use the special command `<user-name> ALL=(ALL) <command-1>, <command-2>`
 - When writing sudo policies, should be aware of the differences between the different Linux and Unix distributions on network. Moreover, some system services have different names on different Linux distributions
 
## Letting users run as other users

 - Change that (ALL) to (root) in order to specify that user can only run these commands as the root user `user ALL=(root) <command-1>, <command-2>`
 - Change ALL=(database) to allow user run as the database user `user ALL=(database) <command-1>, <command-2>` or simply run with this command `sudo -u database <command>`

## Preventing abuse via user's shell scripts

 - If user allow in sudo rules to run own shell script (in home directory where user has full access control here) (`user ALL=(ALL) <shell-script-path>`), being the sneaky type, user can add the `sudo -i` ( log a person in to the root user's shell) line to the end of the script like this, excute it then user now logged in as the root user:
 ```sh
   #!/bin/bash
   echo "Hello World"
   sudo -i
 ```
 - Put shell script `/usr/local/sbin` directory and change the ownership to the root user so user can not change file, then `visudo` and change user rule to reflect the new location of the script

## Detecting and deleting default user account

 - `Internet of Things (IoT)` devices challenge normal operating system installation. In default setting, credentials are out there for all the world to see, and user is set up with full sudo privileges and isn't required to enter a sudo password (Ex. `raspex`)
 - Inside `/etc/sudoers` file of `raspex`: `raspex ALL=(ALL) NOPASSWD: ALL` allows the raspex user to do all sudo commands without having to enter a password
 - Some Linux distributions for IoT devices have this rule in a separate file in the /etc/sudoers.d directory which can be deleted by default user account, or change the root user password, and then lock the root user account

---

## Locking down users' home directories the Red Hat or CentOS way 

 - By default, the `useradd` utility on Red Hat-type systems creates user home directories with a permissions setting of 700 (user who owns the home directory
can access it)
 - In `/etc/login.defs` file of Red Hat-type, there is `UMASK` (set to such a restrictive value by default) line is what determines the permissions values on home directories
   ```
      CREATE_HOME yes
      UMASK 077 # removes all permissions from the group and others
   ```
 - Non-Red Hat distributions usually have a UMASK value of 022, which creates home directories with a permissions value of 755 allowing everybody to enter everybody else's home directories and access each others' files

---

## Locking down users' home directories the Debian/Ubuntu way

Debian/Ubuntu have two user creation utilities

## useradd on Debian/Ubuntu

 - `sudo useradd -m -d /home/user -s /bin/bash user` create a user account (-m creates the home directory, -d home directory path, -s specifies user's default shell)
 - Change the default permissions setting for home directories, open /etc/login.defs for editing `UMASK 022` to `UMASK 077` to lockdown like Red Hat-type

## adduser on Debian/Ubuntu

 - Is an interactive way to create user accounts and passwords with a
single command which is unique to the Debian family but default permission value is 755
 - useradd doesn't automatically encrypt a user's home directory as creating the account. Install the ecryptfs-utils package to solve problem
 - The first time login, using ecryptfs-unwrap-passphrase command to encrypt with passphrase

## Hands-on lab for configuring adduser

```sh
  #!/bin/bash
  # Install package
  PKG_OK=$(dpkg-query -W --showformat='${Status}\n' ecryptfs-utils|grep "install ok installed")
  echo Checking for ecryptfs-utils: $PKG_OK
  if [ "" = "$PKG_OK" ]; then
    echo "No ecryptfs-utils. Setting up ecryptfs-utils."
    sudo apt-get --yes install ecryptfs-utils
  fi

  # Create a user account with an encrypted home directory
  if ((`grep -c cleopatra /etc/passwd` > 0));
    then
      echo "Account: cleopatra exists";
    else
      sudo adduser --encrypt-home cleopatra
      ls -l /home
      su - cleopatra
      ecryptfs-unwrap-passphrase
      exit
  fi;

```

---

## Enforcing strong password criteria

 - Some experts disagree on the details of regular criteria (using complex passwords that regularly expire) cause making password hard to remember and change regularly. 
 - Using passphrases that are long, yet easy to remember 

## Installing and configuring pwquality

 - `pwquality` module for the `Pluggable Authentication Module (PAM)` replaced the old `cracklib` module
 - Providing a way to configure the default password quality requirements for the system passwords. This file is read by the libpwquality library and utilities that use this library for checking and generating passwords
 - /etc/pam.d directory includes PAM configuration files.
 ![](https://i.ibb.co/gSpjGyH/Screenshot-2023-03-01-135409.png)
 - /etc/security/pwquality.conf file includes rest of the procedure, has a very `simple name = value` format with possible comments starting with # character
 - sudo privilege to set user's password, the system will complain if you create a password that doesn't meet complexity criteria, but it still work. Otherwise, normal user were to try to change own password without sudo privileges, the system would not allow a password that doesn't meet complexity criteria

## Hands-on lab for setting password complexity criteria

```sh
  #!/bin/bash
  # Install package
  PKG_OK=$(dpkg-query -W --showformat='${Status}\n' libpam-pwquality|grep "install ok installed")
  echo Checking for libpam-pwquality: $PKG_OK
  if [ "" = "$PKG_OK" ]; then
    echo "No libpam-pwquality. Setting up libpam-pwquality."
    sudo apt-get --yes install libpam-pwquality;
  fi;

  sed 's/# minlen/ minlen = 19/'/etc/security/pwquality.conf # Set minimum of length password
  sed 's/# minclass/ minlen = 19/'/etc/security/pwquality.conf # Required classes of characters for the new password (digits, uppercase, lowercase, others)
  sed 's/# maxclassrepeat/ maxclassrepeat = 5/'/etc/security/pwquality.conf # Repeat time per class


```

## Setting and enforcing password and account expiration

 - Ensure that temporary user accounts aren't forgotten about when they're no longer needed
 - When password expires, user can change it, and everything will be all good. If account expires, only user with the proper admin privileges can unlock it
 - Set password and account expiration data for other users or use the -l option to view expiration data
 ![](https://i.ibb.co/hH7TssB/Screenshot-2023-03-01-141237.png)
 - Everything here is set according to
the out-of-the-box system default values
    - Password inactive
    - Minimum number of days between password change
    - Number of days of warning before password expires
 - /etc/login.defs defines the site-specific configuration for the shadow password suite

## Configuring default expiry data for useradd for Red Hat or CentOS only

 - The /etc/default/useradd file has the rest of the default settings 
 ![](https://i.ibb.co/yfm1b3Q/Screenshot-2023-03-01-143541.png)
 - Using command line `useradd -D` (use alone to see new config) with the appropriate option switch for the item (Ex. `sudo useradd -D -e 2023-12-31` to set a default expiration date of December 31, 2023)

## Setting expiry data on a per-account basis with useradd and usermod

 - Set account expiry data on a per-account basis with 3 methods:
   - `useradd` with the appropriate option  switches (-e: expiration date, -f: number of days after the user's password expires)
   - `usermod` modify expiry data (same `useradd`)
   - `chage --expiredate/--maxdays ` modify expiry data
 - Using `chage -l` to see all new changes

## Setting expiry data on a per-account basis with chage

 - Using chage to modify existing accounts with options

 ![](https://i.ibb.co/4MCynjs/Screenshot-2023-03-01-144752.png)
 ![](https://i.ibb.co/7rpZ6hr/Screenshot-2023-03-01-144857.png)

 - Force user to change password the first time user login with two ways to do. Either way, this would do it after setting that user password initially:
 ```sh
   sudo chage -d 0 <user-name1> or sudo passwd -e <user-name>
 ```

## Hands-on lab for setting account and password expiry data

```sh
  #!/bin/bash
  # Create a user account for Samson with the expiration date of June 30, 2023
  if ((`grep -c samson /etc/passwd` > 0));
    then
      echo "Account: samson exists";
    else
      if ((`grep -c Ubuntu /etc/os-release` > 0));
        then
          sudo useradd -m -d /home/samson -s /bin/bash -e 2023-06-30;
        else if ((`grep -c CentOS /etc/os-release` > 0))
          then
            sudo useradd -e 2023-06-30 samson;
      fi;
      sudo chage -l samson
  fi;
  
  # Use usermod to change Samson's account expiration date to July 31, 2023
  sudo usermod -e 2023-07-31
  sudo chage -l samson

  # Force Samson to change his password on first login
  sudo passwd samson
  sudo passwd -e samson
  sudo chage -l samson

  # Use chage to set a five days waiting period for changing passwords, a password expiration period of 90 days, an inactivity period of two days, and a warning period of five days

  sudo chage -m 5 -M 90 -I 2 -W 5 samson
  sudo chage -l samson

  su - samson
  exit

```

## Preventing brute-force password attacks

 - Possible for early man to brute-force someone
else's password with random number 
 - Nowadays, with strong passwords, or better yet, a strong passphrase, setting a lockout value of three failed login attempts will do three things:
    - Unnecessarily frustrate users
    - Cause extra work for help desk personnel
    - If an account really is under attack, it will lock the account before gathering information about the attacker

## Configuring the pam_tally2 PAM 

 - The `pam_tally2` module comes already installed on both CentOS and Ubuntu, editing the /etc/pam.d/login
 - Some option in file:
    - `deny=4`: lock out after only four failed login attempts 
    - `even_deny_root`: the root user account will get locked if it's under attack
    - `unlock_time=1200`: automatically unlocked after 1,200 (20 minutes)
 -  `pam_tally2` to manually unlock a locked account 
 ![](https://i.ibb.co/Cwmpxvx/Screenshot-2023-03-01-152109.png)

## Hands-on lab for configuring pam_tally2

```sh
  #!/bin/bash
  if ((`grep -c Ubuntu /etc/os-release` > 0));
    then
      sed -i '32 i auth required pam_tally2.so deny=4 even_deny_root unlock_time=1200' /etc/pam.d/login;
    else if ((`grep -c CentOS /etc/os-release` > 0))
      then
        sed -i '2 i auth required pam_tally2.so deny=4 even_deny_root unlock_time=1200' /etc/pam.d/login;
  fi;
  sudo pam_tally2

  # Reset
  sudo pam_tally2 --user=samson --reset
  sudo pam_tally2

```

## Locking user accounts

 - Some reason manual lock account:
    - When a user goes on vacation and you want to ensure that nobody monkeys around with that user's account while this user is gone
    - When a user is under investigation for questionable activities
    - When a user leaves the company
 - There are two utilities that you can use to temporarily lock a user account
    - Using `usermod`
    - Using `passwd`

## Using usermod to lock a user account

 - `sudo usermod -L <user-name>` lock user account
 - In user's entry /etc/shadow file, there is an exclamation point in front of password hash preventing the system from being able to read password
 ![](https://i.ibb.co/2Zc0mmX/Screenshot-2023-03-01-153547.png)
 - `sudo usermod -U <user-name>` unlock account to remove exclamation point

## Using passwd to lock user accounts

 - `sudo passwd -l <user-name>` lock user account
 - Place two exclamation points in front of the password hash, instead of just one like `usermod`
 - `sudo passwd -u <user-name>` unlock account

## Locking the root user account

 - The cloud is big business nowadays, rent a virtual private server from companies so they have logging in to the root user account
 - First thing setup a cloud-based server is to create a normal user account and set it up with full sudo privileges, then using command `sudo passwd -l root`

---   

## Setting up security banners

At any rate, just to be on the safe side, user want to set up login messages that make clear that only authorized users are allowed to access the system

## Using the motd file

 - /etc/motd (stands for `Message of the Day`) file will present a message banner to anyone who logs in to a system through Secure Shell

## Using the issue file

 - A default issue file would just contain macro code that would show information about the machine like, it would show up after a reboot: 
 ```
   Ubuntu 18.04 LTS \n \l
 ```
 - For desktop machines that are out in the open, this would be more useful

## Using the issue.net file

It's for `telnet` logins, and anyone who has telnet enabled on their servers is seriously screwing up and the `issue.net` file still hangs around in the /etc directory

## Detecting compromised passwords

 - One of the most effective ways of brute-forcing passwords is to use these dictionaries to perform a dictionary attack
 - Password-cracking tool reads in passwords from a specified dictionary and tries each one until either the list has been exhausted, or until the attack is successful
 - Instead sending your production password to somebody's website, it just send a hash value of the password 
 - Application Programming Interface (API) and using `curl` for the basic principle: `curl https://api.pwnedpasswords.com/range/21BD1`

## Hands-on lab for detecting compromised passwords

```sh
  #!/bin/bash
  # Install curl on Ubuntu
  PKG_OK=$(dpkg-query -W --showformat='${Status}\n' curl | grep "install ok installed")
  echo Checking for curl: $PKG_OK
  if [ "" = "$PKG_OK" ]; then
    echo "No curl. Setting up curl."
    sudo apt-get --yes install curl;
  fi;

  # Find  how many passwords there are with the 21BD1 string in password hashes
  curl https://api.pwnedpasswords.com/range/21BD1

  # Check password TurkeyLips is compromised
  candidate_password="TurkeyLips"
  echo "Candidate password: $candidate_password"
  full_hash=$(echo -n $candidate_password | sha1sum | awk '{print
  substr($1, 0, 32)}')
  prefix=$(echo $full_hash | awk '{print substr($candidate_password, 0, 5)}')
  suffix=$(echo $full_hash | awk '{print substr($candidate_password, 6, 26)}')
  if curl https://api.pwnedpasswords.com/range/$prefix | grep -i $suffix;
    then echo "Candidate password is compromised";
    else echo "Candidate password is OK for use";
  fi

```

---

## Understanding centralized user management

A way to manage computers and users from one central location, settle for a high level overview

## Microsoft Active Directory

Possible to add Unix/Linux computers and their users to an Active Directory domain

## Samba on Linux

 - Serve three purposes:
    - Share directories from a Unix/Linux server with
Windows workstations
    - Set up as a network print server
    - Set up as a Windows domain controller
 - Install Samba version 3 on a Linux server, and set it up to act as an old-style Windows NT domain controller

## FreeIPA/Identity Management on RHEL/CentOS

 - FreeIPA (IPA: Identity - Policy - Audit) as a set of packages for Fedora
 ![](https://i.ibb.co/BZ6876f/Screenshot-2023-03-02-083142.png)
 - Although adding Windows machines to a FreeIPA domain is posible, it's not recommended. But, starting with RHEL/CentOS 7.1, posible to use FreeIPA to create cross-domain trusts with an Active Directory domain
 - FreeIPA also known as Identity Management or IdM