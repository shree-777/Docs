.. meta::
  :description: Firewall Network Workflow
  :keywords: GCP Transit Gateway, Aviatrix Transit network, Transit DMZ, Egress, Firewall, Firewall Network, FireNet, GCP FireNet


=========================================================
Transit FireNet Workflow for GCP
=========================================================

Aviatrix Transit FireNet allows you to deploy firewalls functions for the Aviatrix Multi-Cloud transit architecture. With Transit FireNet feature, the Firewall Network (FireNet) function is integrated into the Aviatrix Transit gateway.

To learn about Transit FireNet, check out `Transit FireNet FAQ. <https://docs.aviatrix.com/HowTos/transit_firenet_faq.html>`_

If you are looking to deploy firewall networks in AWS Transit Gateway (TGW) environment, your starting point is `here. <https://docs.aviatrix.com/HowTos/firewall_network_workflow.html>`_.


In this example, Transit VPC with Aviatrix Gateways will be deployed, and two Spoke Gateways (DEV and PROD) will be attached to it.

The transit VPC will have a firewall of supported vendors (Checkpoint, Palo Alto Networks and Fortinet etc.) deployed in it. Please see the diagram below for more details.

Once the infra is in-place then the policy will be created to inspect the east-west and north-south traffic.


|avx_tr_firenet_topology|


Step 1 : Create VPCs
***************************

VPCs can be created manually on GCP or directly from Aviatrix Controller.

Aviatrix controller has set of useful tools available for users and in this example, VPCs are created following the Useful Tools `Create a VPC <https://docs.aviatrix.com/HowTos/create_vpc.html>`_ guidelines.

1.	Login to the Aviatrix Controller with username and password
#.	Navigate to **Useful Tools -> Create A VPC**
#.	Add one VPC for Transit FireNet Gateway and provide **Aviatrix FireNet VPC Subnet** as shown below.
#.  Add three more VPCs as shown in Topology i.e Egress VPC, LAN VPC and Management VPC.
#.  Create two more VPCs for Spoke Gateways.

|create_vpc|

Step 2: Deploy the Transit Aviatrix Gateway
***************************************************

Transit Aviatrix Gateway can be deployed using the `Transit Gateway Workflow <https://docs.aviatrix.com/HowTos/transitvpc_workflow.html#launch-a-transit-gateway>`_

Procedure
~~~~~~~~~~~~~~~~~~~~~

1.	Navigate to **MULTI-CLOUD TRANSIT -> Setup -> #1 Launch an Aviatrix Transit Gateway**
#.  Select the Cloud Type **Gcloud**
#.  Select VPC ID **Transit FireNet VPC**
#.	Choose instance size **n1-standard-1**
#.	Enable **ActiveMesh Mode (Mandatory)**
#.  Check **Enable Transit FireNet Function** checkbox and provide LAN VPC ID **Transit FireNet LAN VPC**
#.	Enable InsaneMode for higher throughputs (optional)
#.  Choose correct Account, Public Subnet and click **Create**
#.	Enable Transit Gateway HA by navigating to **MULTI-CLOUD TRANSIT -> Setup -> #2 (Optional) Enable HA to an Aviatrix Transit Gateway**

Please see an example below for Transit FireNet GW:

|tr_firenet_gw|

Step 3: Deploy Spoke Gateways
*************************************

Now that we have Aviatrix Transit Gateway, we can deploy Aviatrix Spoke Gateways in the spoke VPCs using `Aviatrix Spoke Gateway Workflow <https://docs.aviatrix.com/HowTos/transitvpc_workflow.html#launch-a-spoke-gateway>`_.

1.	Navigate to **MULTI-CLOUD TRANSIT -> Setup -> #4 Launch an Aviatrix Spoke Gateway**
#.	Deploy a Spoke Gateway (GW) in each of the spoke VPCs using defaults while choose correct Account and VPC info
#.	Choose the Public Subnet
#.	Enable Spoke Gateway HA by navigating to **MULTI-CLOUD TRANSIT -> Setup -> #5 (Optional) Enable/Disable HA at Spoke GW**

|launch_spk_gw|

Step 4: Attach Spoke Gateways to Transit Network
*******************************************************

Transit and spoke gateways are deployed, next step is to connect them.

1.	Navigate to **MULTI-CLOUD TRANSIT -> Setup -> #6a Attach Spoke Gateway to Transit Network**
#.	Select one spoke at a time and attach to the Transit Gateway.

|attach_spk_trgw|

.. note::
 Transit Gateway is now attached to Spoke Gateways but Transit Gateway will not route traffic between Spoke Gateways.

Step 5: Enable Connected Transit
**************************************

By default, spoke VPCs are in isolated mode where the Transit will not route traffic between them. To allow the Spoke VPCs to communicate with each other, we need to enable Connected Transit

1.	Navigate to **MULTI-CLOUD TRANSIT -> Advanced Config**, select the right Transit Gateway and enable **“Connected Transit”**

|connected_transit|

Step 6: Configure Transit Firewall Network
**************************************************

Transit and Spoke Gateways have now been deployed, next step is to deploy and enable the Firewall for traffic inspection.

Let’s start with enabling the firewall function and configure the FireNet policy.

1.	Navigate to **MULTI-CLOUD TRANSIT -> Transit FireNet -> #1 Enable Transit FireNet on Aviatrix Transit Gateway**
#.	Choose the Aviatrix Transit Gateway and Click **“Enable”**

.. Note::

  For GCP deployment, Transit FireNet function is enabled when launching the gateway, skip this step.


3.	Navigate to **MULTI-CLOUD TRANSIT -> Transit FireNet -> #2 Manage FireNet Policy**
#.	Add spokes to the Inspected box for traffic inspection

.. note::
    By default, FireNet inspects ingress (INET to VPC) and east-west traffic (VPC to VPC) only.

|tr_firenet_policy|


Step 7a: Launch and Associate Firewall Instance
*****************************************************************

This approach is recommended if this is the first Firewall instance to be attached to the gateway.

This step launches a Firewall instance and associates it with one of the FireNet gateways.


.. important::

    The Firewall instance and the associated Aviatrix FireNet gateway above must be in the same AZ, and, we recommend that the Management Interface Subnet and Egress (untrust dataplane) Interface Subnet should not be in the same subnet.

7a.1 Launch and Attach
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Go to Aviatrix Controller's console and navigate to **Firewall Network -> Setup -> Step 7a** and provide all the required input as shown in a table and click **"Launch"** button.

.. important::
    Vendor's firewall may take some time after launch to be available.


==========================================      ==========
**Setting**                                     **Value**
==========================================      ==========
VPC ID                                          The Security VPC created in Step 1.
Gateway Name                                    The primary FireNet gateway.
Firewall Instance Name                          The name that will be displayed on GCP Console.
Firewall Image                                  The AWS AMI that you have subscribed in Step 2.
Firewall Image Version                          Firewall instance current supported software versions.
Firewall Instance Size                          Firewall instance type.
Management Interface VPC ID                     Select the Firewall Management VPC
Management Interface Subnet                     Select the subnet for Firewall Management
Egress Interface VPC ID                         Select the Firewall Egress VPC.
Egress Interface Subnet                         Select the subnet for Firewall Egress.
Attach (Optional)                               By selecting this option, the firewall instance is inserted in the data path to receive packet. If this is the second firewall instance for the same gateway and you have an operational FireNet deployment, you should not select this option as the firewall is not configured yet. You can attach the firewall instance later at Firewall Network -> Advanced page.
Advanced (Optional)                             Click this selection to allow Palo Alto firewall bootstrap files to be specified.
Bootstrap Bucket Name                           In advanced mode, specify a bootstrap bucket name where the initial configuration and policy file is stored.
==========================================      ==========

1. Check Point Specification
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Check Point support for Google Cloud is coming in future release


2. Palo Alto VM-Series Specifications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Palo instance has 3 interfaces as described below.

========================================================         ===============================          ================================
**Palo Alto VM instance interfaces**                             **Description**                          **Inbound Security Group Rule**
========================================================         ===============================          ================================
nic0                                                             Egress or Untrusted interface            Allow ALL
nic1                                                             Management interface                     Allow SSH, HTTPS, ICMP, TCP 3978
nic2                                                             LAN or Trusted interface                 Allow ALL (Do not change)
========================================================         ===============================          ================================

Note that firewall instance nic2 is on the same subnet as FireNet gateway nic1 interface.

.. important::

    For Panorama managed firewalls, you need to prepare Panorama first and then launch a firewall. Check out `Setup Panorama <https://docs.aviatrix.com/HowTos/paloalto_API_setup.html#managing-vm-series-by-panorama>`_.  When a VM-Series instance is launched and connected with Panorama, you need to apply a one time "commit and push" from the Panorama console to sync the firewall instance and Panorama.

.. Tip::

    If VM-Series are individually managed and integrated with the Controller, you can still use Bootstrap to save initial configuration time. Export the first firewall's configuration to bootstrap.xml, create an IAM role and Bootstrap bucket structure as indicated above, then launch additional firewalls with IAM role and the S3 bucket name to save the time of the firewall manual initial configuration.


Follow `Palo Alto Network (VM Series) GCP Example <https://docs.aviatrix.com/HowTos/config_paloaltoGCP.html>`_ to launch VM Series firewall in GCP and for more details.


3. Fortigate Specifications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Fortinet support for Google Cloud is coming in future release


Step 7b: Associate an Existing Firewall Instance
*******************************************************

This step is the alternative step to Step 8a. If you already launched the firewall (Check Point, Palo Alto Network or Fortinet) instance from AWS Console, you can still associate it with the FireNet gateway.

Go to Aviatrix Controller's console and navigate to **Firewall Network -> Setup -> Step 7b** and associate a firewall with right FireNet Gateway.


Step 8: Vendor Firewall Integration
*****************************************************

Vendor integration programs RFC 1918 and non-RFC 1918 routes in firewall appliance.

1.  Login to Aviatrix Controller's console
#.  Go to Firewall Network -> Vendor Integration -> Select Firewall, fill in the details of your Firewall instance.
#.	Click Save, Show and Sync.

Step 9: Example Setup for "Allow All" Policy
***************************************************

After a firewall instance is launched, wait for 5 to 15 minutes for it to come up. Time varies for each firewall vendor.
In addition, please follow example configuration guides as below to build a simple policy on the firewall instance for a test validation that traffic is indeed being routed to firewall instance.

Palo Alto Network (PAN)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For basic configuration, please refer to `example Palo Alto Network configuration guide <https://docs.aviatrix.com/HowTos/config_paloaltoVM.html>`_.

For implementation details on using Bootstrap to launch and initiate VM-Series, refer to `Bootstrap Configuration Example <https://docs.aviatrix.com/HowTos/bootstrap_example.html>`_.


Step 10: Verification
***************************

There are multiple ways to verify if Transit FireNet is configured properly:

    1.	Aviatrix Flightpath - Control-plane Test
    #.	SSH, SCP or Telnet Test between Spoke VPCs (East-West) - Data-plane Test

.. note::
    ICMP is blocked on Google Cloud Load balancer

Flight Path Test for FireNet Control-Plane Verification:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Flight Path is a very powerful troubleshooting Aviatrix tool which allows users to validate the control-plane and gives visibility of end to end packet flow.

    1.	Navigate to **Troubleshoot-> Flight Path**
    #.	Provide the Source and Destination Region and VPC information
    #.	Select SSH and Private subnet, and Run the test

.. note::
    VM instance will be required in GCP, and SSH/Telnet port should be allowed in firewall rules for Spoke VPCs.

SSH/Telnet Test for FireNet Data-Plane Verification:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once control-plane is established and no problem found in security and routing polices. Data-plane validation needs to be verified to make sure traffic is flowing and not blocking anywhere.

There are multiple ways to check the data-plane. One way to SSH to Spoke instance  (e.g. DEV1-VM) and telnet other Spoke instance (e.g PROD1-VM) to make sure no traffic loss in the path.


.. |subscribe_firewall| image:: transit_firenet_workflow_media/transit_firenet_AWS_workflow_media/subscribe_firewall.png
   :scale: 35%

.. |en_tr_firenet| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/en_tr_firenet.png
   :scale: 35%

.. |tr_firenet_policy| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/tr_firenet_policy.png
   :scale: 35%

.. |avx_tr_firenet_topology| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/avx_tr_firenet_topology.png
   :scale: 35%

.. |create_vpc| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/create_vpc.png
   :scale: 35%

.. |tr_firenet_gw| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/tr_firenet_gw.png
   :scale: 35%

.. |launch_spk_gw| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/launch_spk_gw.png
   :scale: 35%

.. |attach_spk_trgw| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/attach_spk_trgw.png
   :scale: 35%

.. |connected_transit| image:: transit_firenet_workflow_media/transit_firenet_GCP_workflow_media/connected_transit.png
   :scale: 35%

.. disqus::
