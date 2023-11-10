Red Hat Enterprise uses firewalld whose rules does not relate back to iptables.  Also firewalld by default blocks all network traffic when it is running.

Reference: [link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-and-configuring-firewalld_configuring-and-managing-networking#predefined-services_getting-started-with-firewalld)

Sample commands:

- sudo systemctl stop firewalld
- sudo firewall-cmd
- sudo firewall-cmd --state
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
- sudo firewall-cmd --remove-port=port-number/port-type

### Add a port and make it permanent
- sudo firewall-cmd --add-port=port-number/port-type
- sudo firewall-cmd --runtime-to-permanent
- sudo firewall-cmd --list-ports
