# Время жизни соеденения, по дефолту 24ч
## /ip firewall connection tracking set tcp-established-timeout=1h

/ip dhcp-server set [find name="defconf"] lease-time=1d
срок аренды dhcp

Отключение не используемых helper'ы (ALG или service port)
/ip firewall service-port
set ftp disabled=yes
set tftp disabled=yes
set irc dissbled=yes
set h323 disabled=yes
set sip disabled=yes
set pptp disabled=yes
set dccp disabled=yes
set sctp disabled=yes