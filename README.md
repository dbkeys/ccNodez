# ccNodez

bash scripts to Refresh DNS zone file with nodes provided through chainz.cryptoid.info API

Uses the chainz.cryptoid.info API, crontab and a bind9 DNS server 
 to enable named vSeed seeders to respond to DNS requests for peer node IP's.
 	This is in lieu of or to complement SIPA node tracker / seeders.

1. Create a DNS zone file template for the delegated sub-domain. Suffix this zone.file name with 'HEAD'.
IP addresses will be appended at the end of it to create the zone file named in /etc/bind/named.conf.local
Please include a serial number sentinel line for easy parsing by dnzf++ :
                ; Serial Number (latest on top)

Example file:  dnsseed7.egulden.org.db.HEAD  ; serves as template.  IP's from chainz.cryptoid.info are added at the end of the file
						     
Ticker:		  	efl
Delegated DNS name:   	dnsseed7
DNS Zone File:		dnsseed7.egulden.org.db
Top Level Domain:	egulden.org

2. get_node_ips <ticker>   ; queries the chainz API and produces a text list of the returned IP's
	Usage: /get_node_ips <ticker> 

3. addips 		   ; appends the IP's to the target zone file
	Usage: addips <IP address list file> <Delegated DNS name> <DNS zone file>

4. dnzf++ 		   ; increments serial number
	Usage: dnzf++ <DNS_zone_file> 

5. Reload zone file into bind9 server; standard bind9 "remote named daemon control" rndc
        rndc reload

6. Automation via crontab is done with the dnzfresh script
	Usage: dnzfresh <delegated_name> <top_level_domain> <ticker>

Example crontab line for egulden, updates zone file every 23 minutes:
*/23 * * * * /usr/local/bin/dnzfresh dnsseed7 egulden.org efl

Example setup of delegated sub-domain "seedz.argentum.cc"

1.  Register domain in /etc/bind/named.conf.local

	zone "seedz.argentum.cc" IN {
		type master;
		file "/var/named/seedz.argentum.cc.db";
	}

2. Create zone file HEAD template
	This bind9 server keeps its zone files in /var/named ...
	/var/named# cat seedz.argentum.cc.db.HEAD 

		;-----------------------------------------------------------------------------------------
		; seedz.argentum.cc     - eGulden DNS node seeder
		;-----------------------------------------------------------------------------------------
		$ORIGIN argentum.cc.
		$TTL    4h
		seedz           IN      SOA     ns2.internam.com.       admin.internam.com. (

				; Serial Number (latest on top)
				1706841776      ;

				25m             ; Slaves:       * refresh every 25 minutes
				12m             ;               * if no answer try back first in 12 minutes
				3d55m           ;               * keep previous info 3 days 55 minutes
				5m )            ;               * keep trying for new info every 5 minutes

				IN      NS      ns2.internam.com.

						; Sending
				IN      TXT     "v=spf1 -all" ; ( Not sending at all. )`

		$ORIGIN argentum.cc.

		; This next section is refreshed periodically via crontab running 'dnzfresh' script
		; obtained from chainz.cryptoid.info via their API, 
		; seedz.argentum.cc
		;-----------------------------------------------------------------------------------------------------

3. Install scripts
	./install
=======

Installation Notes
---------------------------------------------------

apt install jq			; for  Debian / Ubuntu
yum install jq			; for  Alma / CentOS / Rocky

Set the zone file folder path in script 'dnzfresh'

Run 'install' script to:
	cp -p addips /usr/local/bin
	cp -p get_node_ips /usr/local/bin
	cp -p dnzfresh /usr/local/bin
	cp -p dnzf++ /usr/local/bin

Ubuntu:
	May have to edit
		/etc/apparmor.d/usr.sbin.named 

	to allow reading / writing in the zone file folder by adding lines like:
		 /var/named/ rw,
		 /var/named/** rw,

	Reload AppArmor:
		service apparmor reload
		apparmor_status



