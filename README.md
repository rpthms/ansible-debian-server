# ansible-server-debian
Ansible Playbook to deploy a Debian Server

This playbook was initially created to easily set up a VPN server within minutes. I've added a few more roles to this playbook since then. You can modify your site.yml file and select the roles which suit your needs.

This playbook currently includes the following roles:
- apt: Install core packages
- system-config: Set up core config files in /etc
- networking: Configure systemd-networkd and enabling IP forwarding
- nftables: Set up a basic nftables ruleset and allow other roles to create firewall rules.
- unattended-upgrades: Automate security upgrades
- postfix: Set up a send-only Postfix SMTP server to receive notifications from the server
- openvpn: Set up the OpenVPN server and configure firewall rules
- rsyslog: Set up a remote syslog server
- user-config: Local user configurations. This role has beek kept empty for you to fill it with your user configurations. 
- nginx: Setup an HTTPS server block for a given hostname. This role will call the certbot role to generate a TLS cert.
- nginx-base: Setup a default HTTP and HTTPS server using the server's FQDN. This role should NEVER be called. Use the nginx role instead.
- znc: Setup an IRC bounder via ZNC
- wireguard: Setup a WireGuard VPN server
- certbot: Generate a TLS cert for the given hostname. Only uses the DNS-01 challenege for domain verification.
- influxdb: Set up a time series database use InfluxDB
- kapacitor: Set up data processing and alert generation using Kapacitor. Kapacitor gets its data from an InfluxDB server.
- monitor: Set up Telegraf to gather data from a host and send it to an InfluxDB server and set up alerting tasks on a remote Kapacitor server.
- grafana: Set up a Grafana server to create dashboards and view the time series data stored on an InfluxDB server

# Configure the playbook
Before you use this playbook, you'll need to do 2 things

## Creating vars.yml
To keep this playbook as generic as possible, I've moved all the variables into a separate file called vars.yml. You will have to create this file by copying `vars.sample.yml` to `vars.yml` and adding the required information. You can also encrypt `vars.yml` using ansible-vault if you so desire.

## Add OpenVPN related files
You will need to add the following files to the OpenVPN role's files directory (roles/openvpn/files). Make sure the files names are the same as given below:

- ca.crt: Your root certificate (CA file)
- crl.pem: Certicate Revocation List
- dh.pem: Diffie Hellman parameters
- server.crt: Your server's certificate (signed by your CA)
- server.key: Your server's private key
- ta.key: OpenVPN static key required for HMAC authentication

All these files can be easily generated using [EasyRSA](https://github.com/OpenVPN/easy-rsa). Check out the Arch Wiki to learn [how to use EasyRSA](https://wiki.archlinux.org/index.php/Easy-RSA).

# Instructions
1. Make sure your SSH public key has been deployed to the root account as the playbook is using root as the `remote_user`. If you'd rather SSH into your own account and use sudo to run the playbook, then you'll have to replace `remote_user` with `become_user` in site.yml
2. Make sure **python-netaddr** is installed on your system, as some of the roles in this playbook use the `ipaddr` filter which needs that package.
3. Edit the inventory (`/etc/ansible/hosts` or a custom inventory file) to add your Debian server's IP or hostname
4. Change the hosts entry in `site.yml` to your Debian server's IP or hostname 
5. Modify the list of roles provided in `site.yml` according to your requirements.
6. Run the playbook using `ansible-playbook site.yml`

To run only a section of the configuration, make use of the --tags flag. Eg:- `ansible-playbook site.yml --tags rsyslog`
