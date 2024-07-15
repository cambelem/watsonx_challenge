# Installing Instana Agent via Helm Chart
Sign in to the Instana UI, and then select an option to display the agent catalog; for example, on the home page, click Deploy Agent.

Note
If you are starting a new trial instance of Instana, the agent catalog is displayed with a prompt to select a host agent to install.
Click the tile Kubernetes - Helm chart

Enter the cluster name and (optionally) the agent zone that you want the cluster to be part of.

The cluster name (<your_cluster_name>) is the customised name of the cluster monitored by this daemonset.

The agent zone (<your_zone_name>) is used to customize the zone grouping displayed on the infrastructure map.

The agent deployment code is updated with the values that you provide. All of the other required parameters are pre-populated in the agent deployment code, which looks like the following:

helm install instana-agent
--repo https://agents.instana.io/helm \
--namespace instana-agent \
--create-namespace \
--set agent.key='<your_agent_key>' \
--set agent.endpointHost='<your_host_agent_endpoint>' \
--set agent.endpointPort=443 \
--set cluster.name='<your_cluster_name>' \
--set zone.name='<your_zone_name>' \
instana-agent

Copy and then run the agent deployment code with Helm 3.

Tip
To configure the installation, you can specify the values on the command line by using the --set flag or can provide a YAML file with your values by using the -f flag.
Note
If you want to deploy the static host agent, set the flag —set agent.image.name=containers.instana.io/instana/release/agent/static in the agent deployment code.
For a detailed list of all the configuration parameters, see Instana Helm Chart.