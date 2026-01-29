Class 1: AI Step-by Step
========================

In this lab, you'll explore a powerful ecosystem of AI tools that work together to create
sophisticated automation workflows and enhance your productivity.

**First**, you'll set up Ollama, a local AI model runner that allows you to run large language models directly
on your machine without relying on cloud services.

**Next**, you'll configure Open WebUI, which provides an intuitive chat interface for interacting with your
local AI models, similar to ChatGPT but running entirely on your own hardware. You'll also integrate an MCP
proxy, mcpo, into Open WebUI and work with a simple MCP server to deploy a basic app service to BIG-IP.

**Then**, you'll discover Fabric, an innovative framework that uses AI to analyze and process various types
of content through specialized "patterns" or prompts designed for specific tasks like summarizing
documents, extracting insights, or generating reports.

**After that**, you'll dig into n8n, a visual workflow automation platform that connects different services
and APIs, enabling you to create automated pipelines that can trigger AI processing based on various events
or data inputs.

**Finally**, you'll configure Open WebUI to route prompts and responses through the F5 Ai Guardrails API for
policy enforcement and threat detection, enabling enterprise-grade safety controls without sacrificing the
flexibility of self-hosted models.

By the end of this lab, you'll have hands-on experience building a complete AI-powered automation system that you can
build out and customize in your own lab and extend for your specific needs.

**Resources**

- `Docker Cheatsheet <resources/docker-cheatsheet.html>`_
- `Ollama Cheatsheet <resources/ollama-cheatsheet.html>`_

Labs
----

.. toctree::
   :maxdepth: 1
   :caption: Contents:
   :glob:

   labinfo/labinfo*
   module*/module*
