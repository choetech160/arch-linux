Wazuh is a great SIEM / XDR agent. There are multiple ways to install the agent on arch. I will not describe here how to run the server

```bash
sudo pacman -S wazuh-agent # can also be installed via yay
```

The agent needs to know how to connect to the server. This will depends on your installation, if it's on WireGuard, public IP or whatever one can come up with. Usually can be found in `/var/ossec/etc/ossec.conf` file.

Their documentation is pretty solid :

[endpoint enrollment](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/enrollment-methods/via-agent-configuration/linux-endpoint.html)

[endpoint deployment](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)

You can then start the agent:

```bash
systemctl enable wazuh-agent
systemctl start wazuh-agent
systemctl status wazuh-agent
```
