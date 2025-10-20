Lab 1.1 - Installing Ollama
===========================

Ollama is a powerful framework that simplifies running and managing large language
models (LLMs) locally or in the cloud. Ollama streamlines the deployment and interaction
with models like LLaMA, Mistral, and others through a lightweight command-line interface
and API. With Ollama, you can quickly spin up language models on your machine, send prompts,
and receive responses—all without relying on external cloud-based services. This makes it
ideal for offline experimentation, privacy-sensitive applications, and edge AI development.
In this lab, you'll explore how to install Ollama and download and run pre-trained models.

Minimum Requirements
--------------------

To run Ollama, you can do so on any modern system, but the more resources you have the better.
Capable GPUs found on gaming PCs would be ideal for larger models, but you can work with
small models on any system with modest specifications.

.. note::

    This OS in this lab is Ubuntu. All of this can be recreated with Mac or Windows with additional
    steps not covered here. Docker is already installed and where compose files are used, they're already
    on the system. It is encouraged to review them before you launch each service.

**Perform these steps from the LLM Server (Web Shell recommended for ease of use)**

In your deployment, click on the **Components** tab, and under **Systems**, click **Access** on the
**LLM Server** and select **WEB SHELL** as shown in the image below. This will launch the shell which
you will use for the remainder of the labs in this module.

.. image:: images/00_llmserver_webshell_interface.png

Install Ollama
--------------

Using Docker simplifies the installation process—or rather, eliminates it entirely. Docker runs a
pre-configured Ollama container, so there's no traditional installation required. However, since our
Ollama server has an NVIDIA T4 GPU, we need to configure Docker to access it before proceeding.

1. Configure Docker for GPU use

.. code-block:: console

    nvidia-ctk runtime configure --runtime=docker

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/# nvidia-ctk runtime configure --runtime=docker
    INFO[0000] Loading config from /etc/docker/daemon.json
    INFO[0000] Wrote updated config to /etc/docker/daemon.json
    INFO[0000] It is recommended that docker daemon be restarted.

2. Go ahead and restart docker

.. code-block:: console

    systemctl restart docker

3. Review the ollama compose file on the LLM Server in the web shell.

.. code-block:: console

    cd /root/ollama
    cat compose.yaml

The output should resemble this:

.. code-block:: docker

    root@ip-10-1-1-5:/# cd /root/ollama
    root@ip-10-1-1-5:/root/ollama# cat compose.yaml
    services:
      ollama:
        image: ollama/ollama
        container_name: ollama
        ports:
          - "0.0.0.0:11434:11434"
        volumes:
          - model_data:/root/.ollama
        environment:
          - OLLAMA_KEEP_ALIVE=-1
          - OLLAMA_MAX_LOADED_MODELS=5
          - OLLAMA_NUM_PARALLEL=4
        networks:
          - llmserver-labnet
        restart: unless-stopped

    volumes:
      model_data:

    networks:
      llmserver-labnet:
        external: true

Note the key details on what image, container name, ports, persistent data volume, and networks are associated.
The environment variables in this case are there to keep the models loaded in memory and allow concurrent
requests. This should improve the wait times in some of the later labs.

4. Run the Ollama compose service.

.. code-block:: console

    docker compose up -d

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker compose up -d
    [+] Running 5/5
     ✔ ollama Pulled                                                                                                                                                                                                                       48.8s
       ✔ 4b3ffd8ccb52 Pull complete                                                                                                                                                                                                         1.5s
       ✔ 98bf6a5ec929 Pull complete                                                                                                                                                                                                         1.7s
       ✔ 33494fde5d21 Pull complete                                                                                                                                                                                                         1.9s
       ✔ 83ea82cf2566 Pull complete                                                                                                                                                                                                        47.5s
    [+] Running 2/2
     ✔ Volume "ollama_model_data"  Created                                                                                                                                                                                                  0.0s
     ✔ Container ollama            Started

5. Check to see if the container is running

.. code-block:: console

    docker ps

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker ps
    CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS          PORTS                                     NAMES
    9343512ad63c   ollama/ollama            "/bin/ollama serve"      8 minutes ago   Up 8 minutes    0.0.0.0:11434->11434/tcp                  ollama
    3cd5cfcd0dd1   docs-nginx-html-server   "/docker-entrypoint.…"   2 months ago    Up 28 minutes   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   docs-nginx-html-server-1

.. warning::

    **Don't mess with that docs-nginx-html-server container...that's your lab guide!**

6. Check to see if Ollama is available by using the curl command

.. code-block:: console

    curl http://localhost:11434

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root# curl http://localhost:11434
    Ollama is running

Recap
-----
You now have the following:

- Docker configured to take advantage of the system GPU
- A Docker container running the Ollama server
- A Docker volume which is used as the file repository to store the models we'll install so they survive container restarts

Next we'll install some models.
