Red Hat Enterprise uses firewalld whose rules does not relate back to iptables.  Also firewalld by default blocks all network traffic when it is running.

Sample commands:

sudo systemctl stop firewalld


sudo firewall-cmd
sudo firewall-cmd --status
sudo firewall-cmd -h
firewall-cmd --list-services
firewall-cmd --permanent --add-port=514/tcp
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-port=514/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-services
firewall-cmd --list
  133  firewall-cmd --list-all-policies
  134  sudo firewall-cmd --list-all-policies
  135  sudo firewall-cmd --list-ports
  136  sudo firewall-cmd --list-protocols
  139  sudo firewall-cmd --status
  140  sudo firewall-cmd
  141  sudo firewall-cmd -h
  142  sudo firewall-cmd --state
  143  sudo firewall-cmd --list-all
