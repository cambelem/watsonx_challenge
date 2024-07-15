# Installing Instana Agent via Linux
Installing the agent by using the one-liner
Requirement
You need root user privileges to install the Instana host agent.
To use the one-liner to install the host agent, complete the following steps:

Sign in to the Instana UI, and then select an option to display the agent catalog; for example, on the home page, click Deploy Agent located at the top right corner.

Note
If you are starting a new trial instance of Instana, the agent catalog is displayed with a prompt to select a host agent to install.
Click the tile Linux - Automatic Installation (One-liner)

Select the packaging option to install a dynamic or static agent, and the runtime options that you want to use. If you change options, then the script parameters are updated.

Note
The script is pre-filled with your agent key and host agent endpoint.
For example, the UI provides the following script to install the agent with the Dynamic packaging option, and with the Azul Zulu 1.8 and Interactive runtime options:

curl -o setup_agent.sh https://setup.instana.io/agent && chmod 700 ./setup_agent.sh && sudo ./setup_agent.sh -a <your_agent_key> -d <your_agent_key> -t dynamic -e <host-agent-endpoint>

(Optional) Select the option to install and start the host agent as a service (only supported for systemd-based systems). This option adds the -s parameter to the script.

Copy and then run the script to install the host agent.