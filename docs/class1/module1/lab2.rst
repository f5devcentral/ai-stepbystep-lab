Lab 1.2 - Running your first models
===================================

Ollama supports a wide range of open-source language models, from smaller, resource-efficient
variants to large-scale models with more advanced capabilities. A key difference between these
models—beyond their size in parameters—is their context window, which defines how much text
the model can consider at once. Smaller models like llama2:7b or mistral often come with context
limits of 4K to 8K tokens, making them fast and lightweight—great for short prompts, embedded
applications, and edge devices. Larger models such as llama2:13b or mixtral typically support
longer context windows, up to 32K tokens or more, allowing them to understand and generate more
coherent responses over longer conversations or documents. When choosing a model in Ollama,
consider both computational requirements and context needs: smaller models are ideal for speed
and portability, while larger models shine in complex, memory-rich tasks like summarization,
code generation, or multi-step reasoning.

Installing Models
-----------------

You can experiment with whatever models you choose. In this lab we'll stay at 8 billion
parameters and below. You can read up on the models on the `Ollama`_ website.

.. _Ollama: https://ollama.com/models

1. The first model we'll install is **tinyllama**. We'll use the ``docker exec ollama`` command to first specify the container we want to run
a command in and then issue the command, which is ``ollama run tinyllama``. This pulls the model from the registry and installs tinyllama in
the container if it's not already installed.

.. code-block:: console

    docker exec ollama ollama run tinyllama

You won't see output with this command as we've already installed this on your behalf, but if we hadn't, it would look like this:

.. code-block:: console

    root@ip-10-1-1-5:/root# docker exec ollama ollama run tinyllama
    pulling manifest
    pulling 2af3b81862c6: 100% ▕██████████████████▏ 637 MB
    pulling af0ddbdaaa26: 100% ▕██████████████████▏   70 B
    pulling c8472cd9daed: 100% ▕██████████████████▏   31 B
    pulling fa956ab37b8c: 100% ▕██████████████████▏   98 B
    pulling 6331358be52a: 100% ▕██████████████████▏  483 B
    verifying sha256 digest
    writing manifest
    success

2. Let's make sure your models are installed in the docker volume instead of within the container.

.. code-block:: console

    docker volume inspect ollama_model_data

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker volume inspect ollama_model_data
    [
        {
            "CreatedAt": "2025-10-15T15:12:49Z",
            "Driver": "local",
            "Labels": {
                "com.docker.compose.config-hash": "d44db9d608057ef0858cf31d91290e2e5da2d12df2f5e97178c87a812cc7bbf6",
                "com.docker.compose.project": "ollama",
                "com.docker.compose.version": "2.38.2",
                "com.docker.compose.volume": "model_data"
            },
            "Mountpoint": "/var/lib/docker/volumes/ollama_model_data/_data",
            "Name": "ollama_model_data",
            "Options": null,
            "Scope": "local"
        }
    ]

Note the Mountpoint attribute. Now list out that directory.

.. code-block:: console

    ls -las /var/lib/docker/volumes/ollama_model_data/_data

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# ls -las /var/lib/docker/volumes/ollama_model_data/_data
    total 20
    4 drwxr-xr-x 3 root root 4096 Oct 15 15:13 .
    4 drwx-----x 3 root root 4096 Oct 15 15:12 ..
    4 -rw------- 1 root root  387 Oct 15 15:13 id_ed25519
    4 -rw-r--r-- 1 root root   81 Oct 15 15:13 id_ed25519.pub
    4 drwxr-xr-x 4 root root 4096 Oct 15 15:32 models

Looking good!

3. Now verify the model is running

.. code-block:: console

    docker exec ollama ollama ps

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker exec ollama ollama ps
    NAME                ID              SIZE      PROCESSOR    CONTEXT    UNTIL
    tinyllama:latest    2644915ede35    1.4 GB    100% GPU     4096       Forever

Notice the processor is 100% GPU. Good news! You can also take a look at the system GPU detail.

.. code-block:: console

    nvidia-smi

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/# nvidia-smi
    Fri Feb  6 20:08:30 2026
    +---------------------------------------------------------------------------------------+
    | NVIDIA-SMI 535.161.08             Driver Version: 535.161.08   CUDA Version: 12.2     |
    |-----------------------------------------+----------------------+----------------------+
    | GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
    |                                         |                      |               MIG M. |
    |=========================================+======================+======================|
    |   0  Tesla T4                       On  | 00000000:00:1E.0 Off |                    0 |
    | N/A   31C    P0              31W /  70W |   5116MiB / 15360MiB |      0%      Default |
    |                                         |                      |                  N/A |
    +-----------------------------------------+----------------------+----------------------+

    +---------------------------------------------------------------------------------------+
    | Processes:                                                                            |
    |  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
    |        ID   ID                                                             Usage      |
    |=======================================================================================|
    |    0   N/A  N/A      5600      C   /usr/bin/ollama                             972MiB |
    +---------------------------------------------------------------------------------------+

4. Let's run a quick test against the model! We'll do this three different ways, a one-shot prompt,
an interactive shell, and with curl via the API.

**The one-shot method**

.. code-block:: console

    echo "Explain post quantum cryptography, briefly, like I'm five" | docker exec -i ollama ollama run tinyllama

The output should resemble something similar to the outputs below. Note that LLMs are not deterministic, so the
output will not match, but should be generally close.

.. code-block:: console

    root@ip-10-1-1-5:/# echo "Explain post quantum cryptography, briefly, like I'm five" | docker exec -i ollama ollama run tinyllama
    Post quantum cryptography is a new type of encryption technology that has emerged in recent
    years to address the limitations and vulnerabilities of classical cryptographic algorithms.
    This emerging method utilizes quantum computing technology to produce stronger cryptographic
    keys, which are much harder to crack than classical algorithms like symmetric key encryption
    (SKE) or public-key cryptography (PKC). Post quantum cryptography is based on the development
    of quantum computers that can perform quantum computations efficiently, and it offers new
    possibilities for creating secure and private communications. The concept of post quantum
    cryptography is designed to address the limitations of classical cryptographic algorithms like
    symmetric key encryption and public-key cryptography by exploiting the fact that quantum
    computing technology could potentially enable faster and more efficient algorithms for
    cryptographic key generation. Post quantum cryptography is an exciting new area of research,
    and it offers several benefits, including improved security, increased efficiency in
    cryptographic operations, and greater privacy for end users.

**The interactive shell**

.. code-block:: console

    docker exec -it ollama ollama run tinyllama

You should be in the interactive shell now with a **>>>** prompt.

.. code-block:: console

    write a haiku about grass

The output should resemble:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker exec -it ollama ollama run tinyllama
    >>> write a haiku about grass
    green blade, soft and calm
    in the garden breeze's embrace
    like a whispered dream come true

Exit the interactive shell

.. code-block:: console

    /bye

**The API**

We'll use curl in this lab to run a prompt against the Ollama API.

.. code-block:: console

    curl http://localhost:11434/api/generate \
          -H "Content-Type: application/json" \
          -d '{
            "model": "tinyllama",
            "prompt": "Why is grass green?",
            "stream": false
          }' | jq .

The output should resemble this (cleaned up for readability):

.. code-block:: console

    {
      "model": "tinyllama",
      "created_at": "2025-10-15T15:43:19.841925387Z",
      "response": "Grass is often seen as being green because of the photosynthesis process that occurs in its leaves...",
      "done": true,
      "done_reason": "stop",
      "context": [...],
      "total_duration": 2688108078,
      "load_duration": 14147836,
      "prompt_eval_count": 39,
      "prompt_eval_duration": 245846309,
      "eval_count": 279,
      "eval_duration": 2427405396
    }

Here's a breakdown of the response data fields:

========================== =============================================================================================================================
Field                      Description
========================== =============================================================================================================================
**model**                  The name of the model used for generation (e.g., ``tinyllama``).
**created_at**             Timestamp indicating when the response was generated (ISO 8601 format).
**response**               The full generated text output from the model, based on the provided prompt.
**done**                   A boolean that confirms the generation process is complete.
**done_reason**            Explains why generation stopped. Common value: ``"stop"`` (typically means end-of-sentence or EOS token).
**context**                An array of token IDs representing the internal context used during generation. Useful for advanced inspection or chaining.
**total_duration**         Total time taken to generate the response (in nanoseconds).
**load_duration**          Time spent loading the model into memory (in nanoseconds). Only shown on first use or if not cached.
**prompt_eval_count**      Number of tokens in the input prompt.
**prompt_eval_duration**   Time spent evaluating the prompt tokens (in nanoseconds).
**eval_count**             Number of tokens generated in the response.
**eval_duration**          Time spent generating the output tokens (in nanoseconds).
========================== =============================================================================================================================

5. We'll need a few more models that we'll use in later labs. Sometimes pulling the models can take quite a while,
so they've been pre-loaded for you. To verify the installed models, run the following command.

.. code-block:: console

    docker exec ollama ollama list

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker exec ollama ollama list
    NAME                   ID              SIZE      MODIFIED
    qwen2.5:7b-instruct    845dbda0ea48    4.7 GB    6 hours ago
    tinyllama:latest       2644915ede35    637 MB    3 months ago
    deepseek-r1:7b         755ced02ce7b    4.7 GB    3 months ago
    codellama:latest       8fdf8f752f6e    3.8 GB    3 months ago
    llama3.2:3b            a80c4f17acd5    2.0 GB    3 months ago
    deepseek-r1:1.5b       e0979632db5a    1.1 GB    3 months ago

Recap
-----
In this lab, you:

- Installed and verified models
- Ran some quick tests to the tinyllama model via the shell and the ollama API

.. note::
    All the subsequent modules will use a variety of the models we've already loaded. It is recommended that you
    pre-load them now so they'll be ready to act on your prompts when you get things set up. Still in your
    **LLM Server web shell**, run the remaining non-custom models with the commands below, which you can paste
    together. You're fine to move on, just make sure to leave this web shell tab open until the models are done loading.

.. code-block:: console

    docker exec ollama ollama run llama3.2:3b
    docker exec ollama ollama run deepseek-r1:1.5b
    docker exec ollama ollama run deepseek-r1:7b
    docker exec ollama ollama run codellama
    docker exec ollama ollama run qwen2.5:7b-instruct

Next we'll customize a model. Time to get creative!