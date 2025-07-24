Lab 3.1 - Installing n8n
========================

n8n is a free and open fair-code licensed node based workflow automation tool that will allow you to easily create AI agents for use with LLMs out of the box. It is very easy to get started with and does not require a GPU.
In this module, we will install n8n in a docker container on our Jumphost and run it, then verify we can acces it via web UI.

Installing n8n
--------------
1. First, we need to create a Docker volume for the n8n data to live on.

.. code-block:: bash

	docker volume create n8n_data

This should give you a nice persistent volume to keep your n8n workflows intact, even if you need to restart your container for some reason.

2. Next, we will issue a single docker command to download the latest n8n container, set the host networking ports, connect to your n8n_data volume and run it!

.. code-block:: bash

	docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n

At this point, n8n should be running and ready for your first access! Go ahead and click the "n8n Interface" button in your deployment and it should open a new browser window that proxies to port 5678 on the jumphost. This will allow you to create your local user, request a community license and configure your first agent... in the next lab.
