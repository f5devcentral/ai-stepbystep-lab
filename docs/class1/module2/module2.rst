Module 2: LLM Tools - Open WebUI, Fabric
========================================

Now that we have our LLMs up and running, let's integrate a graphical front-end with
Open WebUI and use the cli-based Fabric framework to optimize some workflows. Details
of each tool we'll take a look at are below.

Open WebUI
----------

Open WebUI is a feature-rich, self-hostable web interface designed to work with various
language models and AI services, most commonly used as a frontend for Ollama and
OpenAI-compatible APIs. It provides a ChatGPT-like conversational interface that runs
entirely on your own infrastructure, giving you full control over your data and
conversations while supporting multiple users, authentication, and administrative features.

Open WebUI also supports Model Context Protocol (MCP) integration, allowing you to extend
your AI assistants with additional tools and capabilities. MCP servers can provide language
models with access to external resources like file systems, databases, APIs, and custom
tools, enabling more powerful and practical AI interactions while maintaining security
through controlled, isolated tool execution.

Practical Usage
~~~~~~~~~~~~~~~

Aside from the learning aspect of this, you could look at using this as a method to make AI
services available to your household. There are interesting benefits this way:

* Save money from subscribing to AI chat services by consolidating individual family members
  subscriptions. It is likely that using an AI service's API will be far cheaper than a single
  subscription, let alone if you have mutiple people in your household using services.
* Keep an eye on what your kids are doing with AI with the ability to monitor their chats.
* Experiment with the latest models quickly as you can switch models quickly. As well, services
  like OpenAI typically will have their latest models available via API before their general
  subscription service.

Fabric
------

Daniel Miessler's Fabric is a powerful open-source framework designed to augment human
intelligence by integrating AI into everyday workflows through a collection of modular,
purpose-built AI "patterns." Rather than being just another AI chat interface, Fabric focuses on
creating standardized, reusable prompts and workflows that solve specific problems across domains
like security analysis, content creation, code review, research, and personal productivity.

Lab Architecture
----------------

Now that we've expanded beyond the single instance from Module 1, let's look at how the connectivity
works between our App Server and the LLM Server. This is what you'll be setting up in Modules 2 & 3.

.. mermaid::

    graph TB
        subgraph AppServer["AppServer"]
            WebUI["Open WebUI<br/>:3000"]
            mcpo["mcpo<br/>:8000"]
            f5mcp["f5mcp<br/>:8081"]
            curlmcp["curlmcp<br/>:8082"]
            n8n["n8n<br/>:5678"]
            Fabric["Fabric<br/>(no port)"]
        end

        subgraph LLMServer["LLMServer"]
            Ollama["Ollama<br/>:11434"]

            subgraph Models["Models"]
                tinyllama["tinyllama"]
                codellama["codellama"]
                deepseek1b["deepseek-r1:1b"]
                deepseek7b["deepseek-r1:7b"]
                llama32["llama3.2:3b"]
            end
        end

        %% Connections from AppServer to LLMServer
        WebUI -->|connects to| Ollama
        n8n -->|connects to| Ollama
        Fabric -->|connects to| Ollama

        %% Local connections within AppServer
        WebUI -->|local| mcpo
        mcpo -->|local| f5mcp
        mcpo -->|local| curlmcp

        %% Ollama connections to models
        Ollama --> tinyllama
        Ollama --> codellama
        Ollama --> deepseek1b
        Ollama --> deepseek7b
        Ollama --> llama32

        %% Styling
        classDef appService fill:#e1f5fe,stroke:#01579b,stroke-width:2px
        classDef llmService fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
        classDef model fill:#fff3e0,stroke:#e65100,stroke-width:1px

        class WebUI,mcpo,f5mcp,curlmcp,n8n,Fabric appService
        class Ollama llmService
        class tinyllama,codellama,deepseek1b,deepseek7b,llama32 model

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
