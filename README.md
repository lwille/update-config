Purpose of this tool
====================
In my home, I have a cable connection which occasionally changes its IP address - occasionally means, it could be once a year or once a month. 
My coworkers use DSL connections, which get a new IP on each reconnect.
On our server, we use IP addresses for access restrictions - root logins are only allowed from "our" IPs, as well as remote connections to the database (for having just one development DB).

So, whenever update-config tool is run, it resolves the IP addresses of some dynamic DNS hostnames and updates the appropriate section of the config files with the new addresses.
As configuration files may have a distinct syntax, the user can specify the format of each config file in a very simple manner.

Configuration example
=====================
The script looks after a BEGIN allowed_hosts ... END allowed_hosts block in each file.
For each entry in the hostnames section, the contents of the block will be replaced by a comment and configuration line specified for each file.

Example hosts_allowed.yml file:
    hostnames:
      fred: fred.homelinux.org
      eric: eric.kicks-ass.net
    files:
      postgres:
        path: '/etc/postgresql/8.3/main/pg_hba.conf'
        comment: "# #{name}/#{host}"
        line: "hostssl\tall\t\tall\t\t#{ip}/32\tmd5"

This example configuration will place the lines
    # eric/eric.kicks-ass.net
    hostssl	all		all		84.123.19.250/32	md5
    # fred/fred.homelinux.org
    hostssl	all		all		118.124.164.267/32	md5
between the # BEGIN .. # END lines of your postgresql access configuration file.

The text has to be enclosed by double quotes.
The following variables can be used (using #{variable}):
   ip
   host
   name
After each line, a newline character is automatically inserted.

Sample crontab entry
====================
Create a crontab entry like this - if you know when IP addresses are likely to change, you can make a crontab that executes the resolve script only at that minute.
E.g.
    32,14 1,5 * * *      /usr/sbin/resolver >> /var/log/resolver.log 2>&1
This will execute the resolver at 1:14 am, 1:32 am, 5:14 am and 5:32 am. This may be neccessary if there are some more people working on your server.
