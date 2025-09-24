# checkmk_agent_pfsense
Here's instructions on how to get Check_MK running on pfSense 2.3+ (the latest verified setup being 2.4.3_1).

Since the original thread is getting old, the Netgate forum recommended a new thread, which would probably be helpful anyway so people don't have to search through all the replies to find the useful stuff. Most of the credit for this configuration goes to @joeclifford and @FJerusalem, with a small update I've added.

For issues getting set up, I would recommend checking out the original thread to see if your issue has already been discussed there.

Get to the firewall's shell (console, SSH, serial, etc.)

Install bash:
```
pkg install bash
```

Create directories:
```
mkdir -p /opt/bin
mkdir -p /opt/etc/xinetd.d
```

Download the latest version of the check_mk_agent for FreeBSD:
```
curl "http://git.mathias-kettner.de/git/?p=check_mk.git;a=blob_plain;f=agents/check_mk_agent.freebsd;hb=HEAD" -o /opt/bin/check_mk_agent
```
Make it executable:
```
chmod +x /opt/bin/check_mk_agent
```

Verify the agent can run:
```
/opt/bin/check_mk_agent
```
Create the service config file using the template below:
```
wget /opt/etc/xinetd.d/check_mk
```

Create the script to ensure the check_mk service will be persistent across system updates:
```
wget /opt/filter_check_mk_cron
```

Make it executable:
```
chmod +x /opt/filter_check_mk_cron
```

Run the script manually (to apply the change and reconfigure filters and services):
```
/opt/filter_check_mk_cron
```

Log in to the pfSense webConfigurator, go to Status > System Logs
You should have a log entry about one new service:
```
Jun 15 08:39:03	xinetd	36043	Reconfigured: new=1 old=3 dropped=0 (services)
```
Or an entry about the check_mk service being reconfigured:
```
Jun 15 09:26:05	xinetd	36043	readjusting service check_mk
```

Go to System > Package Manager, and install the "Cron" package if necessary

Go to Services > Cron > Add, and add a cron job to make the script run every 15 minutes:
*/15
*
*
*
*
root
/opt/filter_check_mk_cron

Add any firewall rules that may be needed for your network environment.

There you have it! You should now be able to point your monitoring server to the LAN address of your firewall to get the status from Check_MK. You can always configure extra network rules and/or port forwarding if you need access on another interface.

Again, thanks to @joeclifford and @FJerusalem for their work on this idea!
