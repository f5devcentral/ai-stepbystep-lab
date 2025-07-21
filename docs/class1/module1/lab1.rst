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
Gaming systems with solid GPUs will be necessary with any models of size, but you can work with
small models on any system without a beast of a system.

.. note:: This OS in this lab is Ubuntu. All of this can be recreated with MAC or Windows but
   you'll need to overcome several hurdles. Docker is already installed but the instructions are
   below for your reference.

Docker

  .. code-block:: bash

     apt update
     apt install -y ca-certificates curl gnupg lsb-release
     mkdir -p /etc/apt/keyrings
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
        | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
     echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
        https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" \
        | tee /etc/apt/sources.list.d/docker.list > /dev/null
     apt update
     apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


Install Ollama
--------------

The benefit of using Docker is the install aspect is a bit of a misnomer. Docker is the engine that
is going to simply run the pre-configured install of Ollama in a container. That said, our Ollama
server has an NVIDIA T4 GPU so we need to configure docker to use it before proceeding.

1. Configure Docker for GPU use

.. code-block:: bash

    root@ip-10-1-1-5:/# nvidia-ctk runtime configure --runtime=docker
    INFO[0000] Loading config from /etc/docker/daemon.json
    INFO[0000] Wrote updated config to /etc/docker/daemon.json
    INFO[0000] It is recommended that docker daemon be restarted.
    root@ip-10-1-1-5:/# sudo systemctl restart docker

.. note:: The NVIDIA toolkit was already pre-configured in the AWS instance we're using.

2. Create a local file resource for Docker so we don't have to re-load models when the container restarts

.. code-block:: bash

    root@ip-10-1-1-5:/# sudo docker volume create model_data
    model_data

3. Run the Ollama container, making sure to reference the volume we created

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker run -d -v model_data:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
    Unable to find image 'ollama/ollama:latest' locally
    latest: Pulling from ollama/ollama
    b08e2ff4391e: Pull complete
    9d52186ca5a2: Pull complete
    d0343d5f73e9: Pull complete
    5343bc64fdab: Pull complete
    Digest: sha256:f478761c18fea69b1624e095bce0f8aab06825d09ccabcd0f88828db0df185ce
    Status: Downloaded newer image for ollama/ollama:latest
    bf7f9f4c88acac99d7930d695768183f1a1f930960181c17b1a360508addcadb

4. Check to see if the container is running

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker ps
    CONTAINER ID   IMAGE           COMMAND               CREATED         STATUS         PORTS                                             NAMES
    bf7f9f4c88ac   ollama/ollama   "/bin/ollama serve"   2 minutes ago   Up 2 minutes   0.0.0.0:11434->11434/tcp, [::]:11434->11434/tcp   ollama

5. Check to see if Ollama is available by using the curl command

.. code-block:: bash

    root@ip-10-1-1-5:/root# curl http://127.0.0.1:11434
    Ollama is running

Recap
-----
You now have the following:

- Docker configured to take advantage of the system GPU
- A Docker container running the Ollama server
- A local file repository for Docker to store the models we'll install so they survive container restarts

Next we'll install a couple models.
