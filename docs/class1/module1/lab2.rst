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
parameters and below. You can read up on the models on the [Ollama](https://ollama.com/search) website.

1. Install tinyllm. We'll use the **docker exec** command to first specify the container we want to run
a command in and then issue the command, which is **ollama run tinyllama**. This installs tinyllama in
the container.


.. code-block:: bash

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

2. Let's check to see if our models are indeed installed locally instead of within the container.

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker volume inspect model_data
    [
        {
            "CreatedAt": "2025-07-21T11:46:15Z",
            "Driver": "local",
            "Labels": null,
            "Mountpoint": "/var/lib/docker/volumes/model_data/_data",
            "Name": "model_data",
            "Options": null,
            "Scope": "local"
        }
    ]

    root@ip-10-1-1-5:/root# ls -las /var/lib/docker/volumes/model_data/_data
    total 20
    4 drwxr-xr-x 3 root root 4096 Jul 21 11:52 .
    4 drwx-----x 3 root root 4096 Jul 21 11:46 ..
    4 -rw------- 1 root root  387 Jul 21 11:52 id_ed25519
    4 -rw-r--r-- 1 root root   81 Jul 21 11:52 id_ed25519.pub
    4 drwxr-xr-x 4 root root 4096 Jul 21 12:15 models

Looking good!

3. Now verify the model is running

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker exec ollama ollama ps
    NAME                ID              SIZE      PROCESSOR    UNTIL
    tinyllama:latest    2644915ede35    1.0 GB    100% CPU     4 minutes from now

4. Let's run a quick test against the model! We don't have an interface yet, so let's use
the curl command to call the API.

.. code-block:: bash

    root@ip-10-1-1-5:/root# curl http://localhost:11434/api/generate \
      -H "Content-Type: application/json" \
      -d '{
        "model": "tinyllama",
        "prompt": "Why is grass green?",
        "stream": false
      }'

When you send a prompt to Ollama's /api/generate endpoint (with "stream": false), you'll
receive the full JSON response with metadata and the generated output. Below is a breakdown
of each field in the response:

.. code-block:: bash

    {
      "model": "tinyllama",
      "created_at": "2025-07-21T12:40:08.26066379Z",
      "response": "Grass, like most plants, is typically green due to the presence of chlorophyll in its cells...",
      "done": true,
      "done_reason": "stop",
      "context": [...],
      "total_duration": 2688108078,
      "load_duration": 14147836,
      "prompt_eval_count": 39,
      "prompt_eval_duration": 245846309,
      "eval_count": 100,
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

5. Let's pull a couple more models before moving on. I'm using deepseek-r1:1.5b and llama3.2:3b

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker exec ollama ollama pull deepseek-r1:1.5b
    pulling manifest
    pulling aabd4debf0c8: 100% ▕██████████████████▏ 1.1 GB
    pulling c5ad996bda6e: 100% ▕██████████████████▏  556 B
    pulling 6e4c38e1172f: 100% ▕██████████████████▏ 1.1 KB
    pulling f4d24e9138dd: 100% ▕██████████████████▏  148 B
    pulling a85fe2a2e58e: 100% ▕██████████████████▏  487 B
    verifying sha256 digest
    writing manifest
    success
    root@ip-10-1-1-5:/root# docker exec ollama ollama pull llama3.2:3b
    pulling manifest
    pulling dde5aa3fc5ff: 100% ▕██████████████████▏ 2.0 GB
    pulling 966de95ca8a6: 100% ▕██████████████████▏ 1.4 KB
    pulling fcc5a6bec9da: 100% ▕██████████████████▏ 7.7 KB
    pulling a70ff7e570d9: 100% ▕██████████████████▏ 6.0 KB
    pulling 56bb8bd477a5: 100% ▕██████████████████▏   96 B
    pulling 34bb5ab01051: 100% ▕██████████████████▏  561 B
    verifying sha256 digest
    writing manifest
    success

5. Now let's verify our installed models

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker exec ollama ollama list
    NAME                ID              SIZE      MODIFIED
    llama3.2:3b         a80c4f17acd5    2.0 GB    16 seconds ago
    deepseek-r1:1.5b    e0979632db5a    1.1 GB    About a minute ago
    tinyllama:latest    2644915ede35    637 MB    46 minutes ago

Recap
-----
In this lab, you:

- Installed and verified models
- Ran a quick test to the tinyllama model via the ollama API

Next we'll customize a model. Time to get creative!