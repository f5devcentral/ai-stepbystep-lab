Lab 3.3 - n8n Multi-Agent Workflow
==================================

By the end of this lab, you will have created a flow that can help organizations to save money on AI spend
by selecting the most appropriate model for a given task. A smaller model will classify a user's prompt
and will proxy the prompt on to another model, based on whether or not the prompt was needing reasoning,
coding or something else and select a model that is designed for the need, rather than relying on the user
understanding which model they should use for which tasks.

.. note::

    This lab uses a two-server setup where models run on a dedicated LLM Server and n8n runs on a separate
    App Server to ensure n8n's performance isn't impacted by model inference.

#. Create a new workflow by clicking the + button in the top left and select **Workflow** from the dropdown:

    .. image:: images/07_new_workflow.png

#. Click the large **+** in the center of the canvas and select **On chat message**:

    .. image:: images/n8n_multiagent_onchat.png

#. Click back to canvas and then type a chat message, hit the send chat in the box, then double click the chat node and pin it.

    .. image:: images/n8n_trigger_chat.png
    .. image:: images/n8n_trigger_chat_pin.png

#. Click back to canvas and then click the **+** on the chat node, then search for **text**, then select the
   **Text Classifier** node.

    .. image:: images/n8n_add_classifier_node.png

#. First we'll add the parameters. Reference the image below, then all the text values for the parameters will follow.

    .. image:: images/n8n_classifier_parameters.png

    #. Drag the **chatInput** block from the left side of the dialog window into the **Text to Classify** textbox.
    #. Click the **Add Category** button and use these values:

        .. code-block:: console

            Category: Reasoning
            Deescription: If reasoning need is indicated by the chat message, this is the category to assign.

    #. Click the **Add Category** button and use the values:

        .. code-block:: console

            Category: Coding
            Deescription: If the chat message indicates a need to code or asks for help with computer languages and scripting languages like iRules, JSON or node.js, assign this category.

    #. Click the **Add Option** button, select the **Allow Multiple Classes To Be True** option, and then click the toggle to enable it.
    #. Click the **Add Option** button, select the **When No Clear Match** option, then select the **Output on Extra, 'Other' Branch** in the drop down list.
    #. Click the **Add Option** button, select the **System Prompt Template** option, then paste the system prompt below:

        .. code-block:: console

            Please classify the text provided by the user into one of the following categories: {categories}, and use
            the provided formatting instructions below: If they explicitly ask for coding help, do not fail and
            classify the message as 'Coding'. If they explicitly ask for reasoning help, do not fail and classify
            the message as 'Reasoning'. Otherwise, send the {{ $json.chatInput }} on to the next agent.

#. Next we'll tackle the classifier's settings. Click the **settings** tab, toggle **Retry on Fail** to enabled, then click **Back to canvas**

    .. image:: images/n8n_classifier_settings.png

#. Click the **+** Model connector on the **Text Classifier** and select the **Ollama Chat Model**.

    .. image:: images/n8n_ollama_model_select.png

#. If you completed the previous n8n lab, use the Ollama account credential in the **Credential to connect with**
   field. If not go back to Lab 3.2 for instructions. Select the deepseek-r1:1.5b model, name the node deepseek-r1:1.5b,
   then click **Back to canvas**.

    .. image:: images/n8n_tc_model_parameters.png

#. Your canvas should look like this now. Click the **+** reasoning connector on the **Text Classifer**, search for **AI**, and select the **AI Agent**.

    .. image:: images/n8n_add_reasoning.png

#. Click the **settings** tab, enable the **Retry on Fail** toggle, rename the node to **Reasoning Agent**, then click **Back to canvas**

    .. image:: images/n8n_reasoning_settings.png

#. Add an Ollama model to the **+** Chat Model connector on the **Reasoning Agent**. We've already covered how to do that above, but the parameters you'll want to set here are **Credential to connect with**, the **Model** which should be **deepseek-r1:7b**, and the node name set to the model name. Then click back to canvas.

#. Your workflow should then look close to this:

    .. image:: images/n8n_reasoning_model.png

#. Now you'll repeat the previous three steps to add the Coding Agent and model. Everything should be the same except the node names and the model, which should be **codellama** in this case. After you add the two nodes, your canvas should look like this:

    .. image:: images/n8n_coding_model.png

#. Now you'll again repeat the previous steps to add the Generalist Agent. Everything should be the same except the model, which you can just drag a line over to the existing deepseek-r1:1.5b. After you add the new Generalist Agent node, your canvas should look like this:

    .. image:: images/n8n_generalist_model.png

#. Test time!! Need to populate a few tests that will work to show off the workflow...TBD!!

Challenges
----------

* Observe prompt engineering impact: Notice how poorly engineered prompts yield lesser results
* Modify the system prompt: Adjust the Text Classifier's system prompt and observe changes
* Swap models: Try different models on your agents and compare results
* Analyze data flow: Watch how different prompt types trigger different agent paths

Recap
-----

You now have a functional multi-model workflow that:

* Automatically classifies user prompts
* Routes requests to specialized models
* Optimizes AI spend by using appropriately-sized models
* Demonstrates intelligent prompt routing architecture

This concludes today's AI Step-by-Step exploration of AI tools. We hope you enjoyed the journey and make sure to
watch and star the two repos we're supporting for this effort:

* `AI Step-by-Step <https://github.com/f5devcentral/AI-stepbystep/>`_ - Guidance for building out home-lab AI tools
* `AI Step-by-Step Lab <https://github.com/f5devcentral/ai-stepbystep-lab/>`_ - The UDF lab guide

If you have any feedback for how we can improve or extend this lab, please contact us at `devcentral@f5.com <mailto:devcentral@f5.com>`_.