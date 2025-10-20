:orphan:

Ollama Cheat Sheet
==================

Installation
------------

Linux/macOS
^^^^^^^^^^^

.. code-block:: console

   # Install Ollama
   curl -fsSL https://ollama.ai/install.sh | sh

   # Start Ollama service
   ollama serve

   # Start Ollama in background
   nohup ollama serve &

Windows
^^^^^^^

.. code-block:: console

   # Download and install from https://ollama.ai/download
   # Or use winget
   winget install Ollama.Ollama

Docker
^^^^^^

.. code-block:: console

   # Run Ollama in Docker
   docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

   # Run with GPU support (NVIDIA)
   docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

Model Management
----------------

Pulling Models
^^^^^^^^^^^^^^

.. code-block:: console

   # Pull a model
   ollama pull MODEL_NAME

   # Pull specific model versions
   ollama pull llama2:7b
   ollama pull llama2:13b
   ollama pull llama2:70b

   # Pull code models
   ollama pull codellama
   ollama pull codellama:7b
   ollama pull codellama:13b

   # Pull other popular models
   ollama pull mistral
   ollama pull mixtral
   ollama pull neural-chat
   ollama pull starling-lm
   ollama pull orca-mini

Listing Models
^^^^^^^^^^^^^^

.. code-block:: console

   # List installed models
   ollama list
   ollama ls

   # Show model information
   ollama show MODEL_NAME
   ollama show llama2:7b

Removing Models
^^^^^^^^^^^^^^^

.. code-block:: console

   # Remove a model
   ollama rm MODEL_NAME
   ollama rm llama2:7b

   # Remove multiple models
   ollama rm model1 model2

Running Models
--------------

Interactive Chat
^^^^^^^^^^^^^^^^

.. code-block:: console

   # Start interactive chat
   ollama run MODEL_NAME
   ollama run llama2

   # Chat with specific model version
   ollama run llama2:13b

   # Exit chat session
   /bye
   # or Ctrl+C

Single Prompt
^^^^^^^^^^^^^

.. code-block:: console

   # Single prompt without interactive mode
   echo "Hello, how are you?" | ollama run llama2

   # Using quotes for complex prompts
   ollama run llama2 "Explain quantum computing in simple terms"

Custom Parameters
^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Run with custom parameters
   ollama run llama2 --temperature 0.8 --top-p 0.9

   # Set context length
   ollama run llama2 --ctx-size 4096

API Usage
---------

REST API Endpoints
^^^^^^^^^^^^^^^^^^

**Generate Completion:**

.. code-block:: console

   # Basic completion
   curl http://localhost:11434/api/generate -d '{
     "model": "llama2",
     "prompt": "Why is the sky blue?"
   }'

   # Streaming response
   curl http://localhost:11434/api/generate -d '{
     "model": "llama2",
     "prompt": "Tell me a story",
     "stream": false
   }'

**Chat API:**

.. code-block:: console

   # Chat completion
   curl http://localhost:11434/api/chat -d '{
     "model": "llama2",
     "messages": [
       {
         "role": "user",
         "content": "Hello!"
       }
     ]
   }'

**Model Information:**

.. code-block:: console

   # List models via API
   curl http://localhost:11434/api/tags

   # Show model info
   curl http://localhost:11434/api/show -d '{
     "name": "llama2"
   }'

Python Integration
^^^^^^^^^^^^^^^^^^

.. code-block:: python

   import requests
   import json

   # Basic completion
   def ollama_generate(prompt, model="llama2"):
       url = "http://localhost:11434/api/generate"
       data = {
           "model": model,
           "prompt": prompt,
           "stream": False
       }
       response = requests.post(url, json=data)
       return response.json()["response"]

   # Chat completion
   def ollama_chat(messages, model="llama2"):
       url = "http://localhost:11434/api/chat"
       data = {
           "model": model,
           "messages": messages,
           "stream": False
       }
       response = requests.post(url, json=data)
       return response.json()["message"]["content"]

   # Usage examples
   result = ollama_generate("Explain machine learning")
   print(result)

   messages = [
       {"role": "user", "content": "Hello!"},
       {"role": "assistant", "content": "Hi! How can I help you?"},
       {"role": "user", "content": "What's the weather like?"}
   ]
   chat_result = ollama_chat(messages)
   print(chat_result)

Modelfile Creation
------------------

Basic Modelfile
^^^^^^^^^^^^^^^

.. code-block:: dockerfile

   # Create custom model from base
   FROM llama2

   # Set custom system message
   SYSTEM "You are a helpful coding assistant."

   # Set parameters
   PARAMETER temperature 0.7
   PARAMETER top_p 0.9
   PARAMETER top_k 40

Advanced Modelfile
^^^^^^^^^^^^^^^^^^

.. code-block:: dockerfile

   FROM codellama:7b

   # Custom system prompt
   SYSTEM """
   You are an expert Python developer. Always provide:
   1. Clean, readable code
   2. Proper error handling
   3. Helpful comments
   4. Best practices
   """

   # Model parameters
   PARAMETER temperature 0.1
   PARAMETER top_p 0.9
   PARAMETER top_k 10
   PARAMETER repeat_penalty 1.1

   # Custom template
   TEMPLATE """{{ if .System }}<|system|>
   {{ .System }}<|end|>
   {{ end }}{{ if .Prompt }}<|user|>
   {{ .Prompt }}<|end|>
   {{ end }}<|assistant|>
   {{ .Response }}<|end|>
   """

Creating Custom Models
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Create model from Modelfile
   ollama create my-model -f Modelfile

   # Create model with custom name
   ollama create coding-assistant -f ./coding-modelfile

   # Run custom model
   ollama run my-model

Model Parameters
----------------

Available Parameters
^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Temperature (creativity/randomness): 0.0 - 2.0
   PARAMETER temperature 0.8

   # Top-p (nucleus sampling): 0.0 - 1.0
   PARAMETER top_p 0.9

   # Top-k (top-k sampling): 1 - 100
   PARAMETER top_k 40

   # Repeat penalty: 0.0 - 2.0
   PARAMETER repeat_penalty 1.1

   # Context size
   PARAMETER ctx_size 4096

   # Number of threads
   PARAMETER num_thread 8

Runtime Parameters
^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Set parameters during run
   ollama run llama2 --temperature 0.5 --top-p 0.8 --top-k 20

Configuration
-------------

Environment Variables
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Set Ollama home directory
   export OLLAMA_HOME=/custom/path

   # Set host and port
   export OLLAMA_HOST=0.0.0.0:11434

   # Enable CORS
   export OLLAMA_ORIGINS=*

   # Set GPU layers (for partial GPU usage)
   export OLLAMA_NUM_GPU=1

Service Management
^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Start Ollama service
   ollama serve

   # Start with custom host/port
   OLLAMA_HOST=0.0.0.0:8080 ollama serve

   # Check if service is running
   curl http://localhost:11434/api/tags

Popular Models
--------------

Text Generation Models
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Llama 2 variants
   ollama pull llama2:7b         # 7 billion parameters
   ollama pull llama2:13b        # 13 billion parameters
   ollama pull llama2:70b        # 70 billion parameters

   # Mistral models
   ollama pull mistral:7b
   ollama pull mixtral:8x7b

   # Other popular models
   ollama pull neural-chat:7b
   ollama pull starling-lm:7b
   ollama pull orca-mini:7b
   ollama pull vicuna:7b

Code Generation Models
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Code-specific models
   ollama pull codellama:7b
   ollama pull codellama:13b
   ollama pull codellama:34b

   # Specialized code models
   ollama pull phind-codellama:34b
   ollama pull wizardcoder:15b

Specialized Models
^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Math and reasoning
   ollama pull wizardmath:7b

   # Instruction following
   ollama pull openorca:7b

   # Uncensored models
   ollama pull dolphin-mistral:7b

Import Custom Models
--------------------

From GGUF Files
^^^^^^^^^^^^^^^

.. code-block:: console

   # Create Modelfile for GGUF import
   echo 'FROM ./model.gguf' > Modelfile
   ollama create my-model -f Modelfile

From Hugging Face
^^^^^^^^^^^^^^^^^

.. code-block:: dockerfile

   # Modelfile for HF model
   FROM hf.co/microsoft/DialoGPT-medium

   SYSTEM "You are a conversational AI."
   PARAMETER temperature 0.7

Troubleshooting
---------------

Common Issues
^^^^^^^^^^^^^

.. code-block:: console

   # Check Ollama status
   ollama list

   # Check service logs
   journalctl -u ollama

   # Restart Ollama service
   sudo systemctl restart ollama

   # Check disk space (models are large)
   df -h

   # Check GPU usage (if applicable)
   nvidia-smi

Memory Management
^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Check model memory usage
   ollama ps

   # Stop running model to free memory
   # (Models auto-unload after inactivity)

   # Force garbage collection
   curl -X POST http://localhost:11434/api/gc

Performance Tips
----------------

Optimization
^^^^^^^^^^^^

.. code-block:: console

   # Use smaller models for faster inference
   ollama pull llama2:7b  # instead of 70b

   # Adjust context size based on needs
   ollama run llama2 --ctx-size 2048  # smaller context

   # Use appropriate temperature
   ollama run llama2 --temperature 0.1  # for factual responses
   ollama run llama2 --temperature 0.8  # for creative responses

Hardware Considerations
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # For CPU-only systems, use smaller models
   ollama pull llama2:7b

   # For GPU systems, check VRAM requirements
   # 7B models: ~4-6GB VRAM
   # 13B models: ~8-12GB VRAM
   # 70B models: ~40+ GB VRAM

Integration Examples
--------------------

Shell Scripts
^^^^^^^^^^^^^

.. code-block:: console

   #!/bin/bash
   # Simple Ollama wrapper script

   MODEL="llama2"
   PROMPT="$1"

   if [ -z "$PROMPT" ]; then
       echo "Usage: $0 'your prompt here'"
       exit 1
   fi

   echo "$PROMPT" | ollama run "$MODEL"

Command Line Tools
^^^^^^^^^^^^^^^^^^

.. code-block:: console

   # Create alias for quick access
   alias ask='ollama run llama2'

   # Usage
   ask "What is the capital of France?"

   # Pipe text files for analysis
   cat document.txt | ollama run llama2 "Summarize this text:"

Web Integration
^^^^^^^^^^^^^^^

.. code-block:: javascript

   // JavaScript fetch example
   async function askOllama(prompt, model = 'llama2') {
       const response = await fetch('http://localhost:11434/api/generate', {
           method: 'POST',
           headers: {
               'Content-Type': 'application/json',
           },
           body: JSON.stringify({
               model: model,
               prompt: prompt,
               stream: false
           })
       });

       const data = await response.json();
       return data.response;
   }

   // Usage
   askOllama('Hello, world!').then(console.log);

Useful Commands Summary
-----------------------

Quick Reference
^^^^^^^^^^^^^^^

.. code-block:: console

   # Essential commands
   ollama serve                    # Start Ollama service
   ollama pull MODEL              # Download model
   ollama run MODEL               # Interactive chat
   ollama list                    # List installed models
   ollama rm MODEL                # Remove model
   ollama show MODEL              # Show model info
   ollama create NAME -f FILE     # Create custom model

   # API endpoints
   POST /api/generate             # Generate completion
   POST /api/chat                 # Chat completion
   GET  /api/tags                 # List models
   POST /api/show                 # Show model info