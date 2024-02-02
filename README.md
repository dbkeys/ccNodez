# ccNodez

bash scripts to Refresh DNS zone file with nodes provided through chainz.cryptoid.info API

 	Uses the chainz.cryptoid.info API, crontab and a bind9 DNS server 
 to enable named vSeed seeders to respond to DNS requests for peer node IP's.
 	This is in lieu of or to compleent SIPA node tracker / seeders.

1. Create a DNS zone file for the delegated sub-domain. Suffix this zone.file name with HEAD
        Ip's will be appended at the end of it to create the zone file named in /etc/bind/named.conf.local
        Must include serial number sentinel line for easy parsing by dnzf++

	Example file:  dnsseed7.egulden.org.db.HEAD  ; serves as template. 
						     ; IP's from chainz.cryptoid.info are added
						     ; at the end of the file
						     
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
