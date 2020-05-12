## Shell

- The websites hints that there is another virtual host on the same box, so we add `hms.htb` to our hosts file
- Browsing to this website shows us a piece of software called OpenEMR
- This software has lots of documented vulnerabilities - https://www.open-emr.org/wiki/images/1/11/Openemr_insecurity.pdf
- Use auth bypass and authenticated SQL injection to dump users table
- Find password hash for `openemr_admin` user, `$2a$05$l2sTLIG6GTBeyBf7TAKL6.ttEwJDmxs9bI6LXqlfCpEcY6VF6P0B.`
- Cracking it with John gives the password as `xxxxxx`
- Now we can log in we can exploit the command injection in the `print_command` functionality, which can be found at http://hms.htb/interface/super/edit_globals.php
- With the command injection we can get a reverse shell as `www-data`

## Privesc

- Enumerate what is listening with `netstat -napt`
- We see something listening on localhost, port 11211. This is memcached
- Connect to it with nc
- Enumerate the keys with `stats items` followed by `stats cachedump <slab id> 100` - more info https://stackoverflow.com/questions/19560150/get-all-keys-set-in-memcached
- See that there are `user` and `passwd` keys. Fetch these values with `get user` and `get passwd`
- This gives us the creds for luffy, `luffy:0n3_p1ec3`
- Logging in as luffy and enumerating processes with `ps auxw` we can see there is a docker container running
- This docker container has `/` mounted at `/mnt/pwn` 
- We get a shell in the docker with `docker exec -it <container id> /bin/bash` which drops us into a root shell
- Now we can access all of the host filesystem as root via `/mnt/pwn/`, so we can cat the user and root flag
