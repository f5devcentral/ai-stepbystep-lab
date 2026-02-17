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
    tinyllama:latest    2644915ede35    645 MB    100% CPU     4096       Forever

Notice the processor is 100% CPU. This is good for the smaller models.

4. Now let's do the same thing for one of our GPU models.

.. code-block:: console

    docker exec ollama-gpu ollama run qwen2.5:7b-instruct-q5_0

.. note::

    The GPU models sometimes will error out on loading. You might have to try to run the model again. If it fails a
    second time, move on for now to step 6. We'll circle back on getting all the models loaded at the end of this lab.

5. Once that completes, check to make sure it's running.

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker exec ollama-gpu ollama ps
    NAME                        ID              SIZE      PROCESSOR    CONTEXT    UNTIL
    qwen2.5:7b-instruct-q5_0    16c4cf552635    5.5 GB    100% GPU     4096       Forever

Good news! You can also take a look at the system GPU detail.

.. code-block:: console

    nvidia-smi

The output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# nvidia-smi
    Thu Feb 12 16:23:56 2026
    +---------------------------------------------------------------------------------------+
    | NVIDIA-SMI 535.161.08             Driver Version: 535.161.08   CUDA Version: 12.2     |
    |-----------------------------------------+----------------------+----------------------+
    | GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
    |                                         |                      |               MIG M. |
    |=========================================+======================+======================|
    |   0  Tesla T4                       On  | 00000000:00:1E.0 Off |                    0 |
    | N/A   31C    P0              33W /  70W |   5417MiB / 15360MiB |      0%      Default |
    |                                         |                      |                  N/A |
    +-----------------------------------------+----------------------+----------------------+

    +---------------------------------------------------------------------------------------+
    | Processes:                                                                            |
    |  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
    |        ID   ID                                                             Usage      |
    |=======================================================================================|
    |    0   N/A  N/A      5230      C   /usr/bin/ollama                            5412MiB |
    +---------------------------------------------------------------------------------------+

6. Let's run a quick test against the models! We'll do this three different ways, a one-shot prompt,
an interactive shell, and with curl via the API.

**The one-shot method**

.. code-block:: console

    echo "Explain post quantum cryptography, briefly, like I'm five" | docker exec -i ollama ollama run tinyllama

The output should resemble something similar to the outputs below. Note that LLMs are not deterministic, so the
output will not match, but should be generally close.

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# echo "Explain post quantum cryptography, briefly, like I'm five" | docker exec -i
    ollama ollama run tinyllama

    Post Quantum Cryptography (PQC) is a new generation of secure communication protocols that are designed to resist
    advanced algorithms and attacks. PQC uses mathematical algorithms called "quantum-resistant" cryptosystems, which
    are mathematically difficult but still practical for real-world applications.

    The idea behind PQC is to create an encryption algorithm that's more secure than previous generations of ciphers
    because it's less susceptible to attacks by quantum computers. Here's how it works: Instead of relying on the
    classical, one-way properties of quantum physics for encryption, modern cryptosystems use mathematical algorithms
    that are known as "quantum-resistant" because they can't be broken using classical computing methods.

    The benefits of PQC compared to traditional symmetric key cryptography (S/K) are:

    1. Greater Security: Quantum computers are designed to solve complex problems faster than classical computers,
    which makes them more powerful for breaking cryptosystems.

    2. Faster Communication: PQC can achieve higher communication rates because it's designed to use "quantum-resistant"
    algorithms that can operate at higher levels of precision, speeding up the time it takes for messages to travel
    across a secure channel.

    3. Improved Data Security: Quantum attacks are more complex than classical ones and require more resources to crack.
    PQC has been designed to be resistant to quantum attacks.

    4. Scalable: PQC can be applied to a range of applications from financial transactions to military communications
    systems, making it suitable for use in real-world scenarios.

    In practice, PQC is still relatively new and researchers are continuing to develop the technology further to ensure
    its security. As with any new technological development, there's ongoing debate about how secure PQC will be against
    quantum attacks or other forms of cyber-attack, but for now it remains a promising approach to securing
    communication channels in real-world situations.

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

We'll use curl in this lab to run a prompt against the Ollama API. (If the qwen model hasn't loaded yet, you can change
the port below to 11434 and the model to tinyllama)

.. code-block:: console

    curl http://localhost:11435/api/generate \
          -H "Content-Type: application/json" \
          -d '{
            "model": "qwen2.5:7b-instruct-q5_0",
            "prompt": "Why is grass green?",
            "stream": false
          }' | jq .

Note I am using the ollama-gpu port of 11435 instead of the default 11434 that is bound to the ollama container.
The output should resemble this (cleaned up for readability):

.. code-block:: console

    {
      "model": "qwen2.5:7b-instruct-q5_0",
      "created_at": "2026-02-12T16:28:50.052694871Z",
      "response": "Grass appears green because of the way its chlorophyll molecules absorb and reflect...",
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
**model**                  The name of the model used for generation (e.g., ``qwen2.5:7b-instruct-q5_0``).
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

7. We'll need a few more models that we'll use in later labs. Sometimes pulling the models can take quite a while,
so they've been pre-loaded for you. To verify the installed models, run the following command.

.. code-block:: console

    docker exec ollama ollama list

You could use ollama-gpu here as well, the list would be the same because all the models are in a shared volume. The
output should resemble this:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker exec ollama ollama list
    NAME                        ID              SIZE      MODIFIED
    qwen2.5:7b-instruct-q5_0    16c4cf552635    5.3 GB    14 minutes ago
    qwen2.5:7b-instruct         845dbda0ea48    4.7 GB    2 weeks ago
    tinyllama:latest            2644915ede35    637 MB    3 months ago
    deepseek-r1:7b              755ced02ce7b    4.7 GB    3 months ago
    codellama:latest            8fdf8f752f6e    3.8 GB    3 months ago
    llama3.2:3b                 a80c4f17acd5    2.0 GB    3 months ago
    deepseek-r1:1.5b            e0979632db5a    1.1 GB    3 months ago

Recap
-----
In this lab, you:

- Installed and verified models
- Ran some quick tests to the tinyllama and qwen2.5 models via the shell and the ollama API

.. note::
    All the subsequent modules will use a variety of the models we've already loaded. It is recommended that you
    pre-load them now so they'll be ready to act on your prompts when you get things set up. Still in your
    **LLM Server web shell**, run the following command in the /root/ollama folder and the remaining non-custom models
    will load in your ollama and ollama-gpu containers while you move on to the next lab. **Do not close out the shell
    while this script runs!**

.. code-block:: console

    ./launch_ollama_lab.sh

This will look similar to the output below.

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# ./launch_ollama_lab.sh
    [INFO] Checking Docker Compose services...
    [INFO] Containers already running, skipping docker compose up

    [INFO] =========================================
    [INFO] Setting up CPU Container (62GB RAM)
    [INFO] =========================================
    [INFO] Waiting for ollama to be ready...
    [INFO] ollama is running!
    [INFO] Waiting for Ollama API at http://localhost:11434 to be responsive...
    [INFO] ollama API is ready!
    [INFO] Verifying and loading models for ollama...
    [INFO] Checking model: tinyllama
    [INFO] ✓ tinyllama is available
    [INFO] ✓ tinyllama already loaded in memory, skipping
    [INFO] Checking model: deepseek-r1:1.5b
    [INFO] ✓ deepseek-r1:1.5b is available
    [INFO] ✓ deepseek-r1:1.5b already loaded in memory, skipping
    [INFO] Checking model: llama3.2:3b
    [INFO] ✓ llama3.2:3b is available
    [INFO] ✓ llama3.2:3b already loaded in memory, skipping
    [INFO] Checking model: deepseek-r1:7b
    [INFO] ✓ deepseek-r1:7b is available
    [INFO] Loading deepseek-r1:7b into memory (this may take several minutes for large models)...
    [INFO] ✓ deepseek-r1:7b loaded successfully (3m 18s)

    [INFO] =========================================
    [INFO] Setting up GPU Container (T4 - 16GB VRAM)
    [INFO] =========================================
    [INFO] Waiting for ollama-gpu to be ready...
    [INFO] ollama-gpu is running!
    [INFO] Waiting for Ollama API at http://localhost:11435 to be responsive...
    [INFO] ollama-gpu API is ready!
    [INFO] Verifying and loading models for ollama-gpu...
    [INFO] Checking model: qwen2.5:7b-instruct-q5_0
    [INFO] ✓ qwen2.5:7b-instruct-q5_0 is available
    [INFO] ✓ qwen2.5:7b-instruct-q5_0 already loaded in memory, skipping
    [INFO] Checking model: codellama
    [INFO] ✓ codellama is available
    [INFO] Loading codellama into memory (this may take several minutes for large models)...
    [INFO] ✓ codellama loaded successfully (9m 3s)

    [INFO] =========================================
    [INFO] Final Verification
    [INFO] =========================================

    [INFO] CPU Container (ollama) - Models:
    [INFO] Verifying models are loaded in ollama...
      ✓ tinyllama (loaded)
      ✓ deepseek-r1:1.5b (loaded)
      ✓ llama3.2:3b (loaded)
      ✓ deepseek-r1:7b (loaded)

    [INFO] GPU Container (ollama-gpu) - Models:
    [INFO] Verifying models are loaded in ollama-gpu...
      ✓ qwen2.5:7b-instruct-q5_0 (loaded)
      ✓ codellama (loaded)
    [INFO] Debug: cpu_ok='true', gpu_ok='true'

    [INFO] 🎉 All models loaded and verified successfully
    [INFO]
    [INFO] CPU Container: http://localhost:11434 (4 models, ~8.8GB RAM)
    [INFO] GPU Container: http://localhost:11435 (2 models, ~11.7GB VRAM)
    [INFO]
    [INFO] Total runtime: 12m 22s
    [INFO] Lab is ready to use


Next we'll customize a model. Time to get creative!