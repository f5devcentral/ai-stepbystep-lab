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

    services:
      ollama-gpu:
        image: ollama/ollama
        container_name: ollama-gpu
        gpus: all
        shm_size: "8gb"
        ports:
          - "0.0.0.0:11435:11434"
        volumes:
          - model_data:/root/.ollama
        environment:
          - OLLAMA_KEEP_ALIVE=-1
          - OLLAMA_MAX_LOADED_MODELS=2
          - OLLAMA_NUM_PARALLEL=1
          - OLLAMA_GPU_OVERHEAD=2147483648
          - OLLAMA_RUNNER_STARTUP_TIMEOUT=600
          - OLLAMA_NUM_GPU=20
          - NVIDIA_VISIBLE_DEVICES=all
        networks:
          - llmserver-labnet
        restart: unless-stopped

      ollama:
        image: ollama/ollama
        container_name: ollama
        shm_size: "8gb"
        ports:
          - "0.0.0.0:11434:11434"
        volumes:
          - model_data:/root/.ollama
        environment:
          - OLLAMA_KEEP_ALIVE=-1
          - OLLAMA_MAX_LOADED_MODELS=4
          - OLLAMA_NUM_PARALLEL=1
          - OLLAMA_RUNNER_STARTUP_TIMEOUT=180
        networks:
          - llmserver-labnet
        restart: unless-stopped

    volumes:
      model_data:

    networks:
      llmserver-labnet:
        external: true

Note the key details on what image, container name, ports, persistent data volume, and networks are associated. Also
note that we are deploying Ollama in two containers, one that utilizes the GPU for the two largest models, and then
another that will be CPU bound only for the other models. This compromise is in place due to the VRAM limitations on
the Tesla T4 GPU. The environment variables in this case are there to take advantage of the GPU where appropriate, keep
the models loaded in memory and allow concurrent requests. This should improve the wait times in some of the later labs.

4. Run the Ollama compose service.

.. code-block:: console

    docker compose up -d

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker compose up -d
    [+] up 2/2
     ✔ Container ollama Created                                                                                                                                                                                        0.1s
     ✔ Container ollama-gpu Created

.. note::

    If this was the first time docker compose was run, the image would need to be pulled and/or built and
    would take more time. In order to prevent network and registry issues with Docker and Ollama, the images
    and models have already been managed.

5. Check to see if the container is running

.. code-block:: console

    docker ps

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker ps
    CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                     NAMES
    5e1e6b679b74   ollama/ollama            "/bin/ollama serve"      37 seconds ago   Up 37 seconds   0.0.0.0:11435->11434/tcp                  ollama-gpu
    85e7672a57c8   ollama/ollama            "/bin/ollama serve"      37 seconds ago   Up 37 seconds   0.0.0.0:11434->11434/tcp                  ollama
    3cd5cfcd0dd1   docs-nginx-html-server   "/docker-entrypoint.…"   6 months ago     Up 4 minutes    0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   docs-nginx-html-server-1

.. warning::

    **Don't mess with that docs-nginx-html-server container...that's your lab guide!**

6. Check to see if your Ollama containers are available by using the curl command

.. code-block:: console

    curl http://localhost:11434
    curl http://localhost:11435

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama#     curl http://localhost:11434
        curl http://localhost:11435
    Ollama is runningOllama is runningroot@ip-10-1-1-5:/root/ollama#

Not the prettiest of outputs, but if you see two "Ollama is running" smashed together, you're good to go!

Recap
-----
You now have the following:

- Docker configured to take advantage of the system GPU
- A Docker container running the Ollama server
- A Docker volume which is used as the file repository to store the models we'll install so they survive container restarts

Next we'll install some models.
