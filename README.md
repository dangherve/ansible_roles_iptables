Iptables
=========

Install netfilter-persistent package and configure iptables


Variables
=========

###### Default policy

``` yml
iptables_v4_filter_input_policy: DROP
iptables_v4_filter_forward_policy: DROP
iptables_v4_filter_output_policy: ACCEPT

iptables_v4_nat_input_policy: ACCEPT
iptables_v4_nat_output_policy: ACCEPT
iptables_v4_nat_prerouting_policy: ACCEPT
iptables_v4_nat_postrouting_policy: ACCEPT
```

``` yml
# Optionnal : defined user defined ipchains
iptables_v4_filter_chain:
  - list of the user defined chains
# End optionnal
```

###### For Filter table

``` yml
iptables_v4_filter_rules:
  - name: "name of the rule" # is used for commentary  (-m comment --comment "name of the rule")
    chain: the name of the chain # default INPUT can be: INPUT, FORWARD, OUTPUT or user defined in iptables_v4_filter_chain
    negate_src: true| false # (negate the src field by adding ! before -s if set to true)
    srcs: list of sources to filter # can be a host or net with CDR notation ( -s )
    negate_dest: true| false # (negate the dest field by adding ! before -d if set to true)
    dest: destination of the packet to filter # can be a host or net with CDR notation ( -d )
    proto: tcp | udp | icmp ... # default tcp if defined (-p)
    icmptype: icmp-type # (0 - echo-reply ; 8 - echo-request ... ) if proto==icmp
    port: # --dport
    ports: # --dports
    module:
      - name: module to used (ctsate, state, log, limit ...)   #  -m
        value: the parameter pass to the module                # --module  
    in: interface in   # (-i )
    out: interface out # (-o )
    target: the target of the chain (when using user defined chain) # (-j target)
```

###### For NAT table


``` yml

iptables_v4_nat_rules:
  negate_src: true| false # (negate the src field by adding ! before -s if set to true)
  srcs: list of sources to filter # can be a host or net with CDR notation ( -s )
  negate_dest: true| false # (negate the dest field by adding ! before -d if set to true)
  dest: destination of the packet to filter # can be a host or net with CDR notation ( -d )
  proto: tcp | udp | icmp ... # default tcp if defined (-p)
  icmptype: icmp-type # (0 - echo-reply ; 8 - echo-request ... ) if proto==icmp
  port: # --dport
  target: DNAT| SNAT | MASQUERADE
  nat: ip:port  # if target=DNAT where to NAT ip:port where to forward the packet and if target=SNAT nat=ip the new source of the source packet

```


Examples:
=========

Basic configuration

1. Web Server
  * Open Web server port 80
  * Open ssh access for 10.0.0.0/8 and 192.168.1.0/24
  * Permit icmp for everyone
  * Internal (via lo) communication allowed

``` yml
iptables_v4_filter_rules:
  - name: INPUT Chain Already established connections module state
    module:
      - name: state
        value: RELATED,ESTABLISHED
  - name: Accept HTTP, HTTPS request
    ports: [80, 443]
    proto: tcp
  - name: Accept from localhost
    in: lo
  - name: Permit ICMP echo request
    proto: icmp
    icmptype: 8
  - name: Accept ssh
    proto: tcp
    port: 22
    srcs:
      - 10.0.0.0/8
      - 192.168.1.0/24
```
