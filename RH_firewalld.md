Red Hat Enterprise uses firewalld whose rules does not relate back to iptables.  Also firewalld by default blocks all network traffic when it is running.

Sample commands:

- sudo systemctl stop firewalld
- sudo firewall-cmd
- sudo firewall-cmd --status
- sudo firewall-cmd -h
- sudo firewall-cmd --list-services
- sudo firewall-cmd --permanent --add-port=514/tcp
- sudo systemctl start firewalld
- sudo firewall-cmd --reload
- sudo firewall-cmd --list-services
- sudo firewall-cmd --list
- sudo firewall-cmd --list-all-policies
- sudo firewall-cmd --list-all-policies
- sudo firewall-cmd --list-ports
- sudo firewall-cmd --list-protocols
- sudo firewall-cmd --status
- sudo firewall-cmd --state
- sudo firewall-cmd --list-all

### Add a port and make it permanent
- sudo firewall-cmd --add-port=port-number/port-type
- sudo firewall-cmd --runtime-to-permanent
- sudo firewall-cmd --list-ports
