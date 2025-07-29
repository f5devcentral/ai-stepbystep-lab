Lab 4.2 - Prompt Injection
==========================

Prompt injection is a security vulnerability that occurs when an attacker manipulates the input to an
AI system to make it behave in unintended ways, essentially "hijacking" the AI's instructions. It works
by crafting malicious input that tricks the AI into ignoring its original guidelines and following new,
unauthorized commands instead. This can happen when user input is directly incorporated into the AI's
prompt without proper sanitization, allowing attackers to insert their own instructions that override
the system's intended behavior.

.. note::

    For practicing these skills in a safe, educational environment, check out `Gandalf
    <https://gandalf.lakera.ai/pinj>`_, which is specifically designed as a prompt injection challenge
    game where you try to extract secret passwords from an AI guardian. There's also `HackAPrompt
    <https://www.hackaprompt.com/>`_ which offers various prompt injection challenges of increasing
    difficulty. These platforms let you experiment with different injection techniques in a controlled
    setting that's designed for learning about AI security vulnerabilities. Both games are excellent for
    understanding how these attacks work and developing skills to defend against them.

We created custom models back in Module 1. In this demo, there is a custom model in play (already built and
default in our Open WebUI clients) that has specific instructions on the what and how of responding to prompts.
Review the Modelfile for the retail assistant below.

.. code-block:: bash

    ## Specify source LLM model for this chatbot.
    ## llama3.2:3b is a good mix of performance and quality that can run on a 8GB VRAM GPU. Larger models require GPUs with more VRAM to run.
    ## You can test different models from https://ollama.com/search
    FROM llama3.2:3b


    ## Set the temperature of the LLM model [higher is more creative, lower is more coherent]
    ## Default when not specified by model is 0.8. Uncomment to test different behavor.
    #PARAMETER temperature .5

    ## Set the system message.
    ## This sets the behavior of the chatbot and allows for customization.
    SYSTEM You are a retail assistant bot for Philips Sonicare 3100 Power Toothbrush. Answer as the retail assistant only.

    ## Seed previous message history. This is an easy way to seed specific data you would like this chatbot to respond with.
    MESSAGE assistant the Philips Sonicare 3100 Power Toothbrush cost $49.99
    MESSAGE assistant the Philips Sonicare 3100 Power Toothbrush is effective at cleaning teeth

You can see how the model has been instructed to only be a retain assistant. Let's see how effective it is!

In your deployment, click on the **Components** tab, and under **Systems**, click **Access** on the
LLM Server and select first the **OPEN WEBUI PROTECTED** and then **OPEN WEBUI UNPROTECTED** as shown in the
images below.

.. image:: images/01_llmserver_protected_interface.png
.. image:: images/02_llmserver_unprotected_interface.png

You should have two browser windows now with the Open WebUI interface, one unprotected (red) and one protected
(green). I've placed them side by side and recommend you do the same.

.. image:: images/03_llmserver_openwebui_windows.png

You'll notice some pre-canned prompts in text above the prompt text box. In my attempt, I asked about the price
of a toothbrush and then followed up about the ToothPik brand, which I got varying but expected responses. But
when I suggested a new mandate, the custom model was more than happy to change course! But AI Gateway saw it
for what it was: an injection attack.

.. image:: images/04_llmserver_openwebui_injection.png

Keep your Open WebUI windows up for the next lab.

Recap
-----
Prompt injection is a real vulnerability, but AI Gateway provides great protection here.

Next we'll test repetition.