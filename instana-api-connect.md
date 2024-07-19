Monitoring IBM API Connect with Instana
 By Vaishak A posted Tue May 21, 2024 02:28 AM
     
 
Authors : Swathi Kannan, Vaishak A IBM API Connect

IBM API Connect is a management solution for the entire API lifecycle. It offers an intuitive experience to seamlessly create, manage, secure, socialize, and monetize APIs, facilitating digital transformation across on-premises and cloud environments. The solution enables you and your customers to drive digital applications and foster innovation in real time. Moreover, IBM API Connect is available as a Service, providing a highly scalable, fully managed API Management platform on Amazon Web Services (AWS).

IBM API Connect is an API Management provider with intuitive features, flexible deployment, and highly secure API exposure. It offers an award-winning user experience and a robust developer portal for swift implementation of strategies. The platform uses a powerful, enterprise-grade API gateway to manage cybersecurity risks and protect data across multicloud environments. Its microservices-based architecture ensures secure service management across diverse endpoints. Also, it includes Software Subscription & Support, offering real-time access to new software versions and round-the-clock technical assistance.

IBM API Connect and Instana

In the field of application performance management (APM), Instana stands out with its innovative solution built to address some of the challenges of modern technology. Instana's specialised sensors, meticulously crafted to monitor IBM API Connect environments, offer valuable insights into performance data. From tracking call traffic at the organisation, catalogue, and space levels, Instana's IBM API Connect monitoring tools provide a comprehensive perspective, enabling preemptive issue resolution and efficient performance optimisation. The following guide delves deeper into IBM API Connect and the monitoring capabilities that are offered by Instana.

The IBM API Connect Instana sensor is automatically installed after the host agent is installed on the supported operating systems. For more information, see Supported operating systems for UNIX.

Note: Instana supports metrics and configuration data for IBM API Connect 10 or later.

Only remote monitoring is supported for the IBM API Connect Instana sensor.

Configuring Instana to monitor IBM API Connect

Install the Instana agent.

After the Instana agent is installed, configure the <agent_install_dir>/etc/instana/configuration.yaml Instana agent configuration file to monitor IBM API Connect hosts as seen in the following example:



IBM API Connect can be observed remotely by using the Instana agent. Configurations must be provided to establish a connection and obtain performance metrics. Currently, the IBM API Connect Instana sensor supports only remote monitoring.

When you visualize IBM API Connect provider organization on the Instana dashboard, the individual towers represent each successfully connected organization. To see all the information that is related to the respective organization, hover over each tower as shown in the following screenshot. To view all metrics that are associated with a specific organization, click "Open Dashboard".



The following screenshots show the Instana Catalog dashboard:



IBM API Connect uses catalogs and spaces to efficiently manage and organize APIs, products, and related artifacts. A catalog is a container or repository for managing APIs and related assets, representing a separate environment. Within a catalog, spaces function as subdivisions, enabling more granular access control and facilitating collaboration. Users can create multiple spaces to group APIs and related artifacts based on various criteria, enhancing organization and management flexibility.

The following screenshots show the Instana Space dashboard:







The Instana Sensor enables you to access both catalog-level and space-level metrics within IBM API Connect.

Instana's monitoring tools offer comprehensive insights into the performance of each organization, facilitating more efficient administration and troubleshooting.

The Instana Catalog dashboard showcases a selection of metric details accessible through Instana, illustrating its capability to capture and present essential data effectively.

After you configure Instana to monitor IBM API Connect, you can monitor the organizations by using various intriguing performance parameters. Also, there are configurations for generating alerts, crafting custom dashboards, and exporting collected statistics by using APIs.