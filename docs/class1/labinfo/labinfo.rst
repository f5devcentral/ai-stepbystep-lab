Lab Information
===============

Lab Topology
------------

- 1 x BIG-IP v17.1.0.1-0.0.4

- 1 x LLM Server

 - Docker

  - NGINX (serving the lab guide)
  - Ollama

- 1 x App Server

 - Docker

  - Open WebUI
  - MCPO w/ F5-mcp & curl-mcp
  - Fabric
  - n8n

Network Addressing
------------------

The following table lists VLANS, IP Addresses, and Credentials for all
components:

.. list-table:: Lab Components
   :widths: 15 30 30 30
   :header-rows: 1

   * - **Component**
     - **Management IP**
     - **VLAN/IP Address(es)**
     - **Credentials**

   * - LLM Server
     - 10.1.1.5
     - | **External:** 10.1.10.5
       | **Internal:** 10.1.20.5; 10.1.20.101-103
     - na

   * - App Server
     - 10.1.1.4
     - | **External:** 10.1.10.4
       | **Internal:** 10.1.20.4
     - na

   * - BIG-IP
     - 10.1.1.6
     - | **External:** 10.1.10.6
       | **Internal:** 10.1.20.6
     - | admin/LTMletmein00!
       | root/LTMletmein00!


Click Next to move on to Module 1.