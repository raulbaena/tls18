Part client de la practica 1 
Creem un fixter anomenat ext.client.conf
'''
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
'''

LO primer que hem de fer es generar una clau rsa amb la seguent conmanda. En aquest cas la clau es diura 'clientvpnkey.pem'
'''
root@raul-laptop:/etc/openvpn# openssl genrsa -out clientvpnkey.pem
Generating RSA private key, 2048 bit long modulus
.............+++
...................+++
e is 65537 (0x010001)
'''
Un cop generada la key generarem  una request per que la nostra CA anomenada VeritataAboluta la firmi
'''
root@raul-laptop:/etc/openvpn#  openssl req -new -key clientvpnkey.pem -out clientvpnreq.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ca
State or Province Name (full name) [Some-State]:barcelona
Locality Name (eg, city) []:barcelona
Organization Name (eg, company) [Internet Widgits Pty Ltd]:edt
Organizational Unit Name (eg, section) []:inf
Common Name (e.g. server FQDN or YOUR name) []:www.test-client.org
Email Address []:admin@edt.org

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:jupiter
An optional company name []:jupiter
'''

Ja feta la request pasem a signar la nostra request amb la nostra CA
'''
root@raul-laptop:/etc/openvpn/client# openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in clientvpnreq.pem -days 3650 -CAcreateserial -extfile ext.client.conf -out clientvpncert.pem
Signature ok
subject=C = ca, ST = barcelona, L = barcelona, O = edt, OU = inf, CN = www.test-client.org, emailAddress = admin@edt.org
Getting CA Private Key
'''

Creem un fitxer de configuracio del client anomenat client.conf amb la seguent configuracio
'''

#
# Sample client-side OpenVPN 2.0 config file #
# for connecting to multi-client server. #
# #
# This configuration can be used by multiple #
# clients, however each client should have #
# its own cert and key files. #
# #
# On Windows, you might want to rename this #
# file so it has a .ovpn extension #
##############################################
# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client
# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun
# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one. On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap
# Are we connecting to a TCP or
# UDP server? Use the same setting as
# on the server.
;proto tcp
proto udp
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote 35.176.19.26 1194 #i27 és el host remot
;remote my-server-2 1194
# Choose a random host from the remote
# list for load-balancing. Otherwise
# try hosts in the order specified.
;remote-random
# Keep trying indefinitely to resolve the
# host name of the OpenVPN server. Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite
# Most clients don't need to bind to
# a specific local port number.
nobind
# Downgrade privileges after initialization (non-Windows only)
;user nobody
;group nobody
# Try to preserve some state across restarts.
persist-key
persist-tun
# If you are connecting through an
# HTTP proxy to reach the actual OpenVPN
# server, put the proxy server/IP and
# port number here. See the man page
# if your proxy server requires
# authentication.
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]
# Wireless networks often produce a lot
# of duplicate packets. Set this flag
# to silence duplicate packet warnings.
;mute-replay-warnings
# SSL/TLS parms.
# See the server config file for more
# description. It's best to use
# a separate .crt/.key file pair
# for each client. A single ca
# file can be used for all clients.
ca cacert.pem
cert clientvpncert.pem
key clientvpnkey.pem
# Verify server certificate by checking that the
# certicate has the correct key usage set.
# This is an important precaution to protect against
# a potential attack discussed here:
# http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the keyUsage set to
# digitalSignature, keyEncipherment
# and the extendedKeyUsage to
# serverAuth
# EasyRSA can do this for you.
remote-cert-tls server
# If a tls-auth key is used on the server
# then every client must also have the key.
;tls-auth ta.key 1
# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
;cipher x
cipher AES-256-CBC
# Enable compression on the VPN link.
# Don't enable this unless it is also
# enabled in the server config file.
comp-lzo # Atenció o tots dos la tene o no la tenen!
# Set log file verbosity.
verb 3
# Silence repeating messages
;mute 20
'''

Un cop crat aquest fitxer executem les seguents comades
'''
systemctl daemon-reload
systemctl start openvpn-client@client.service
'''

Fem un ping pero comprovar que client es comunica amb el servidor y viceversa
'''
root@raul-laptop:/etc/openvpn/client# ping 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=66.3 ms
64 bytes from 10.8.0.1: icmp_seq=2 ttl=64 time=56.4 ms
64 bytes from 10.8.0.1: icmp_seq=3 ttl=64 time=171 ms
64 bytes from 10.8.0.1: icmp_seq=4 ttl=64 time=47.5 ms
64 bytes from 10.8.0.1: icmp_seq=5 ttl=64 time=38.5 ms
64 bytes from 10.8.0.1: icmp_seq=6 ttl=64 time=222 ms
64 bytes from 10.8.0.1: icmp_seq=7 ttl=64 time=88.0 ms
64 bytes from 10.8.0.1: icmp_seq=8 ttl=64 time=39.9 ms
64 bytes from 10.8.0.1: icmp_seq=9 ttl=64 time=42.0 ms
64 bytes from 10.8.0.1: icmp_seq=10 ttl=64 time=67.5 ms
^C
--- 10.8.0.1 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9014ms
rtt min/avg/max/mdev = 38.596/84.060/222.906/59.440 ms
'''

Tambe comporvem que la tarja de xarxa virtual anomenada tun s'hagi creat
'''
8: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::567e:55aa:3e:63d6/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
'''

Tambe comporvem el broadcast
'''
root@raul-laptop:/etc/openvpn/client# ip r
default via 192.168.1.1 dev wlan0 proto dhcp metric 600 
10.8.0.1 via 10.8.0.5 dev tun0 
10.8.0.5 dev tun0 proto kernel scope link src 10.8.0.6 
169.254.0.0/16 dev wlan0 scope link metric 1000 
'''

 Comprovem que fem ping del servidor al client
'''
 [fedora@ip-172-31-19-108 tmp]$ ping 10.8.0.6  
PING 10.8.0.6 (10.8.0.6) 56(84) bytes of data.
64 bytes from 10.8.0.6: icmp_seq=1 ttl=64 time=71.6 ms
64 bytes from 10.8.0.6: icmp_seq=2 ttl=64 time=105 ms
64 bytes from 10.8.0.6: icmp_seq=3 ttl=64 time=45.5 ms
64 bytes from 10.8.0.6: icmp_seq=4 ttl=64 time=65.7 ms
64 bytes from 10.8.0.6: icmp_seq=5 ttl=64 time=42.1 ms
^C
--- 10.8.0.6 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 42.190/66.148/105.598/22.740 ms
'''
Generar la part servidor y posar-lo en marxa
'''
$ openssl genrsa -out serverkey.vpn.pem
$ openss req -new -key serverkey.vpn.pem -out serverreq.vpn.pem
$ openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in serverreq.vpn.pem -days
3650 -CAcreateserial -extfile ext.server.conf -out servercert.vpn.pem
'''

Generar la part client y posar-lo en marxa
'''
$ openssl genrsa -out clientkey.1vpn.pem
$ openssl req -new -key clientkey.1vpn.pem -out clientreq.1vpn.pem
$ openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in clientreq.1vpn.pem -days
3650 -CAcreateserial -extfile ext.client.conf -out clientcert.1vpn.pem
# systemctl start openvpn-client@client.service
'''
