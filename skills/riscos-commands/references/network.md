# RISC OS Network Operations

* `*AddFS`
  * `*AddFS <station number> [<disc number> [:]<disc name>]`
  * *AddFS adds the given file server and disc name to those NetFS currently knows about. If only the station number is given then that station will be removed from the list of known file servers.

* `*AddMap`: *AddMap <IP address> <net> adds an entry to the Net address map

* `*AppleAARPCache`: *AppleAARPCache shows the contents of the AARP cache.

* `*AppleFS`: *AppleFS selects AppleTalk as the current filing system.

* `*ApplePAPStatus`
  * `*ApplePAPStatus [-interface <interface name>] -address <socket>.<network>.<node ID> | [-name] <object>[:<type>][@<zone>]`
  * *ApplePAPStatus gives the status of the AppleTalk printer. Default 'type' is "LaserWriter".

* `*ApplePAPTalk`
  * `*ApplePAPTalk [-interface <interface name>] -address <socket>.<network>.<node ID> | [-name] <object>[:<type>][@<zone>]`
  * *ApplePAPTalk goes into interactive mode with given PostScript� printer. Default 'type' is
  * "LaserWriter".

* `*ApplePing`
  * `*ApplePing [-interface <interface name>] -address <socket>.<network>.<node ID> | [-name] <object>[:<type>][@<zone>]
  * *ApplePing sends AppleTalk AEP packets to the specified destination node. You should receive replys
  * when the destination node has a working AppleTalk stack with AEP layer and the network is fully
  * functioning. When no interface is specified, ApplePing will take the first available one. AEP socket  is defined as 4.

* `*AppleStatus`: *AppleStatus gives the AppleTalk protocol stack status for each found network interface.

* `*AppleZones`
  * `*AppleZones [-interface <interface name>]`
  * *AppleZones gives all AppleTalk zones defined on the Internet. When no interface is specified,
  * ApplePing will take the first available one.

* `*DHCP`: *DHCP <interface> starts a DHCP session on that interface.

* `*DHCPStatus`: *DHCPStatus displays information about all DHCP sessions.

* `*DelMap`: *DelMap <net> | <IP Address> removes an entry from the Net address map

* `*FwShow`: *FWShow displays all currently known Freeway objects

* `*IFConfig`
  * `*IfConfig [-ea] <interface> [<address> [<dest_addr>]] [<parameters>]`
  * Command is: IFConfig        2.10 (20 Sep 2006)
  * 
  * *IFConfig is used to configure internet interfaces.
  * 
  * Switches:
  *   -e  write error report to Inet$Error.
  *   -a  list all present interfaces.
  * 
  * Parameters:
  *   up               Mark an interface up
  *   down             Mark an interface down
  *   arp              Enable the use of ARP
  *   -arp             Disable the use of ARP
  *   metric <n>       Set the routing metric to <n> (default 0)
  *   netmask <mask>   Set the interface netmask to <mask>
  *   broadcast <addr> Set the network broadcast to <addr>
  *   alias <addr>     Set an additional interface address
  *   delete <addr>    Remove an interface address
  *   mtu <n>          Set the interface maximum transmission unit to <n>

* `*InetStat`
  * `*INetStat [-Aan] [-f address_family] [-M core] [-N system]`
  * Command is: INetStat        1.07 (13 Oct 2005)
  * 
  * *INetStat displays status information about the Internet module.
  *         *INetStat [-bdghimnrs] [-f address_family] [-M core] [-N system]
  *         *INetStat [-bdn] [-I interface] [-M core] [-N system] [-w wait]
  *         *INetStat [-M core] [-N system] [-p protocol]

* `*InternetIP`
  * *Configure InternetIP allows the configuration of the first ethernet interface on machine startup.
  * Valid values are 'none', 'aun', 'static <address>/<prefix>' and 'dynamic'. These may be followed by
  * a DNS description ('dns <address>' or 'dns auto'), and a default route ('gw <address>' or 'gw auto').

* `*Logon`
  * `*Logon [:<File server name>|[:]<station number>] <user name> [[:<CR>]<Password>]`
  * *Logon initialises the current (or given) file server for your use. Your user name and password are
  * checked by the file server against the password file.

* `*Net`: *Net selects the network as the current filing system.

* `*NetIAutoMap`
  * *Configure NetIAutoMap allows the configuration of whether NetI configures the AUN map to match the interfaces.

* `*NetMap`: *NetMap [<net>] displays the current Net address map

* `*NetProbe`: *NetProbe <station_number> checks that a remote station is alive

* `*NetStat`: *NetStat [-a] displays current address and status information

* `*ResolverConfig`: *ResolverConfig re-reads the resolver configuration from the Inet$ variables.

* `*Route`
  * `*Route [<switches>] add|delete|change|get [-net|-host] <dest> [<gateway>] [flags]`
  * Command is: Route           2.08 (13 Oct 2005)
  * 
  * *Route is used to manipulate the Internet routing tables.
  *         *Route [<switches>] flush|monitor
  * 
  * <dest> is destination host or network
  * <gateway> is next-hop gateway
  * 
  * Switches:
  *   -cloning    generates a new route on use
  *   -iface      destination is directly reachable
  *   -static     manually added route
  *   -nostatic   pretend route added by kernel or daemon
  *   -reject     emit an ICMP unreachable when matched
  *   -blackhole  silently discard pkts (during updates)
  *   -rtt <n>    initial round-trip time
  *   -rttvar <n> initial RTT variance
  *   -mtu <n>    initial MTU
  *   -expire <n> expiry time
  * 
  * Options:
  *   -e          write error report to Inet$Error
  *   -n          show network addresses as numbers
  *   -v          (verbose) print additional details
  *   -q          suppress all output

* `*RouterDiscovery`
  * *RouterDiscovery [-host | -router | -remove] <interface> controls the use of ICMP Router Discovery
  * on an interface.

* `*SysLog`
  * `*SysLog <log name> <priority> [<message>]`
  * *SysLog will change the logging levels, log a message or change the logging configuration.
  *         *SysLog <log name> SHOW | SHOWCACHE
  *         *SysLog SERVER | ON | OFF

* `*SysLog_Flush`
  * `*SysLog_Flush [ON|OFF]`
  * *SysLog_Flush with no options will flush all currently open log files to disc. By default full
  * flushing is disabled. In this state, logged messages will only be written to the background after a
  * short period - the write to disc is delayed. To enable full flushing, and therefore force all data
  * to be written immediately, use the 'ON' option. To disable again, use the 'OFF' option.

* `*SysLog_PollPeriod [<period>]`: *SysLog_PollPeriod will display, or change, how often remote logging is checked and logs flushed.

* `*SysLog_Status`
  * `*SysLog_Status [<log name> | *]`
  * *SysLog_Status will give the status of all the logs currently known to SysLog, or the status of a single log. The name `*` can be used to display the default log parameters.
