Lab 3.2 - n8n Hello World
=========================

This lab relies on the Ollama instance you built in Module 1, so please ensure that your Ollama container is running on 10.1.1.5 before continuing.

.. code-block:: bash

   docker ps

You should see an output similar to this:

.. code-block:: bash

   root@ip-10-1-1-5:/# docker ps
   CONTAINER ID   IMAGE           COMMAND               CREATED       STATUS       PORTS                                             NAMES
   e9d5e7367dab   ollama/ollama   "/bin/ollama serve"   7 hours ago   Up 7 hours   0.0.0.0:11434->11434/tcp, [::]:11434->11434/tcp   ollama

Goals
======

By the end of this short lab, you will have created your first AI agent. It doesn't do a whole lot, except to proxy the conversation between n8n's native chat
interface and your LLM served by Ollama, but it will clearly display the power that an agent provides and it will set you up for easy AI-powered automation.

Steps
=====

#. Now, it's time to open your deployment's n8n Interface Access Method and create an owner account for the instance:

   .. image:: images/00_n8n_Interface.png

#. Enter account details and click **Next**.

   .. image:: images/01_owner_account.png

#. This form is optional. You can click **Get Started** without filling in details to move on.

   .. image:: images/02_customize_screen.png

#. You can request your free community license activation key by entering an email address.

   .. image:: images/03_send_license.png

#. Click **usage and plan** to enter your license key that should be in your email.

   .. image:: images/04_usage_plan.png

#. Click the Enter activation key button and paste your key from your email:

   .. image:: images/03_license_email.png
   .. image:: images/05_plan_screen.png
   .. image:: images/06_key_pasted.png

#. Create a new workflow by clicking the + button in the top left of the screen and select Workflow from the drop-down:

   .. image:: images/07_new_workflow.png

#. Trigger your flow with a chat message. Click the big + in the center of the canvas and select On chat message:

   .. image:: images/08_add_trigger.png

#. We don't need to configure anything on the next screen so click on **Back to canvas** to return to the canvas:

   .. image:: images/08_back_to_canvas.png

#. Create your agent! Click the + icon that is attached to your trigger and select AI > AI Agent from the side menu. When your agent definition screen comes up,
take a look around, but change nothing and head back to your canvas:

   .. image:: images/09_click_ai.png
   .. image:: images/10_select_agent.png
   .. image:: images/11_agent_define.png

#. Enable your agent with Ollama. Click the add chat model button, then select Ollama Chat Model from the side menu. When the Ollama definition screen comes up,
click in the box and create new credentials. All you need to do for Ollama credentials is to set the actual IP of your host machine by replacing ``localhost`` with
``10.1.1.5`` Once complete, close out of the definition screen:

   .. image:: images/12_connect_model.png
   .. image:: images/13_create_cred.png
   .. image:: images/14_add_ip.png
   .. image:: images/15_close_out.png

#. TEST IT OUT!! Click **Open chat**. Type "Hello World" in the chat box and watch in amazement as you proxy your first chat conversation through an agent:

   .. image:: images/16_open_chat.png
   .. image:: images/16_type_hello.png
   .. image:: images/17_mic_drop.png

#. Homework: Now that you've done the lab, explore the memory and tool options in your agent. The memory allows you to insert your chat data in any of a number of databases. The tools are connectors to various other resources and web utilities like ticketing services and chats. Check out the various triggers besides chat interface, as well. There are incredible ways to trigger these flows, too. Please imagine the possibilities for automating a million things in your workday with simple agentic flows.
