Lab 1.3 - Customizing a Model
=============================

Ollama uses a simple, Dockerfile-like syntax called a ``Modelfile`` to define customized models.
This file allows you to build on top of an existing base model and apply instructions or
configuration changes to control how the model behaves. The ``Modelfile`` is declarative and
supports directives such as ``FROM`` to set the base model, ``SYSTEM`` to define default behavior
or tone, ``PARAMETER`` to adjust inference settings, and ``TEMPLATE`` to control prompt formatting.
You can also embed files using the ``COPY`` command to preload context into the model’s working memory.

Common Modelfile directives include:

- ``FROM``: The base model to build on (e.g., ``mistral`` or ``llama2``)
- ``SYSTEM``: Default system instructions (e.g., “You are a helpful coding assistant.”)
- ``PARAMETER``: Controls like ``temperature``, ``top_p``, or ``stop`` sequences
- ``TEMPLATE``: Custom prompt templates for consistent request/response structure
- ``COPY``: Embeds local files for contextual grounding

Examples:

.. code-block:: docker

   FROM mistral
   SYSTEM You are a friendly assistant that answers in simple, easy-to-understand language.
   PARAMETER temperature 0.3

This example sets up a customized version of the ``mistral`` model with a calm, simple
tone and more deterministic behavior.

.. code-block:: docker

   FROM llama2
   COPY ./faqs.txt /docs/faqs.txt
   SYSTEM You are a support bot that answers questions based only on the company FAQ in /docs/faqs.txt.
   PARAMETER temperature 0.2
   TEMPLATE """{{ .System }}

   FAQ: {{ .Prompt }}

   Answer:"""

This version creates a grounded model that responds based on a local FAQ file, with a
structured template to guide prompt formatting.

Using a ``Modelfile`` in this way helps create focused, reliable AI tools for customer
support, internal automation, or creative writing—without needing full model fine-tuning.

Copying Modelfile into a Named Docker Volume
--------------------------------------------

If you're using a Docker *named volume* (e.g., ``model_data``) for Ollama storage, you'll need
to copy your custom model files (like a ``Modelfile`` or supporting context files) into the
volume before you can build the model inside the container. One way to do this is by using a
temporary helper container.

Step-by-Step Instructions Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: This section is for review of the process to create your custom model. I'll provide an example
afterward, but I encourage you to get creative and define your own.

**Create your model directory on the host:**

.. code-block:: bash

   mkdir /tmp/my-custom-models
   cd /tmp/my-custom-models
   cat > Modelfile <<EOF
   FROM tinyllama
   SYSTEM You are a helpful assistant that answers clearly.
   PARAMETER temperature 0.3
   EOF

.. note:: If you want to create multiple models, our project directory might look like this below.
   Each subdirectory contains the `Modelfile` and any related context files for a specific model.

.. code-block:: text

   my-custom-models/
   ├── coding-bot/
   │   └── Modelfile
   ├── summarizer/
   │   ├── Modelfile
   │   └── reference.txt
   └── support-bot/
       ├── Modelfile
       └── faqs.txt

**Use a temporary container to copy files into the volume:**

.. code-block:: bash

   docker run --rm \
     -v model_data:/root/.ollama \
     -v /tmp/my-custom-models:/tmp/my-custom-models \
     alpine sh -c "cp -r /tmp/my-custom-models/* /root/.ollama/models/"

This command:
- Mounts the Docker volume where Ollama stores model data
- Mounts your custom models folder
- Recursively copies each model folder into ``/root/.ollama/models`` inside the volume

**Create the models using ``docker exec``:**

.. code-block:: bash

   docker exec ollama ollama create -f /root/.ollama/models/coding-bot/Modelfile -t coding-bot
   docker exec ollama ollama create -f /root/.ollama/models/summarizer/Modelfile -t summarizer
   docker exec ollama ollama create -f /root/.ollama/models/support-bot/Modelfile -t support-bot

.. note:: These are examples that do not have Modelfiles unless you created them. We'll step through
   a complete example down below.

**Run your custom model:**

Once built, each model can be run independently

.. code-block:: bash

   docker exec ollama ollama run summarizer "Summarize this article: [text]"
   docker exec ollama ollama run support-bot "How do I reset my password?"

Best Practices
~~~~~~~~~~~~~~

- Keep model folders well-organized and named clearly
- Include versioning or metadata inside each ``Modelfile`` if needed
- Automate the copy/build process with a shell script or `make` task if you're working with many models regularly

Creating a Custom Model
~~~~~~~~~~~~~~~~~~~~~~~

In this lab, we'll create a couple custom models. Both will format responses to your prompt in the form of dialogue
between characters. The first between Calvin & Hobbes based on the tinyllama model, and the second between Phineas
and Ferb based on the deepseek R1 1.5b model.


#. Create the Modelfiles in a temporary folder structure on your ollama host.

.. code-block:: bash

    mkdir -p /tmp/custom-models/calvin-hobbes
    mkdir -p /tmp/custom-models/phineas-ferb

    cat > /tmp/custom-models/calvin-hobbes/Modelfile <<EOF
    FROM llama3.2:3b

    SYSTEM You are generating a short, imaginative, and clever dialogue between Calvin and Hobbes from the comic strip. Calvin is a curious, mischievous child with a wild imagination. Hobbes is his thoughtful, sarcastic, and loyal tiger friend. Their conversations are humorous, philosophical, and often explore big ideas through a childlike lens.

    TEMPLATE "Calvin: {{ .Prompt }}\nHobbes:"

    PARAMETER temperature 0.9
    PARAMETER top_p 0.95
    PARAMETER stop "\n\n"
    EOF

    cat > /tmp/custom-models/phineas-ferb/Modelfile <<EOF
    FROM llama3.2:3b

    SYSTEM You are generating a short, witty dialogue between Phineas and Ferb from the animated series. Phineas is energetic, imaginative, and always ready to build something wild. Ferb is quiet, clever, and typically delivers a short, insightful or funny comment. Their conversations should feel like the setup to one of their inventive summer projects.

    TEMPLATE "Phineas: {{ .Prompt }}\nFerb:"

    PARAMETER temperature 0.85
    PARAMETER top_p 0.9
    PARAMETER stop "\n\n"
    EOF

#. Verify your models in /tmp/custom-models

.. code-block:: bash

    ls -alsR /tmp/custom-models

The output should resemble:

.. code-block:: bash

    root@ip-10-1-1-5:/tmp# ls -alsR /tmp/custom-models
    /tmp/custom-models:
    total 24
     4 drwxr-xr-x  4 root root  4096 Jul 23 14:37 .
    12 drwxrwxrwt 12 root root 12288 Jul 23 14:32 ..
     4 drwxr-xr-x  2 root root  4096 Jul 23 14:25 calvin-hobbes
     4 drwxr-xr-x  2 root root  4096 Jul 23 14:25 phineas-ferb

    /tmp/custom-models/calvin-hobbes:
    total 12
    4 drwxr-xr-x 2 root root 4096 Jul 23 14:25 .
    4 drwxr-xr-x 4 root root 4096 Jul 23 14:37 ..
    4 -rw-r--r-- 1 root root  473 Jul 23 14:25 Modelfile

    /tmp/custom-models/phineas-ferb:
    total 12
    4 drwxr-xr-x 2 root root 4096 Jul 23 14:25 .
    4 drwxr-xr-x 4 root root 4096 Jul 23 14:37 ..
    4 -rw-r--r-- 1 root root  487 Jul 23 14:25 Modelfile

#. Load the custom model modelfiles to your model_data docker volume by creating a temporary container

.. code-block:: bash

    docker run --rm \
      -v model_data:/root/.ollama \
      -v /tmp/custom-models:/tmp/custom-models \
      alpine sh -c "cp -r /tmp/custom-models/* /root/.ollama/models/"

#. Verify the modelfiles are now in your model_data docker volume

.. code-block:: bash

    ls -als /var/lib/docker/volumes/model_data/_data/models

The output should resemble:

.. code-block:: bash

    root@ip-10-1-1-5:/root# ls -als /var/lib/docker/volumes/model_data/_data/models
    total 24
    4 drwxr-xr-x 6 root root 4096 Jul 23 14:47 .
    4 drwxr-xr-x 3 root root 4096 Jul 23 14:44 ..
    4 drwxr-xr-x 2 root root 4096 Jul 23 14:46 blobs
    4 drwxr-xr-x 2 root root 4096 Jul 23 14:47 calvin-hobbes
    4 drwxr-xr-x 3 root root 4096 Jul 23 14:45 manifests
    4 drwxr-xr-x 2 root root 4096 Jul 23 14:47 phineas-ferb

#. Create your custom models

.. code-block:: bash

    docker exec ollama ollama create phineas-ferb -f /root/.ollama/models/phineas-ferb/Modelfile
    docker exec ollama ollama create calvin-hobbes -f /root/.ollama/models/calvin-hobbes/Modelfile

The output should resemble:

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker exec ollama ollama create phineas-ferb -f /root/.ollama/models/phineas-ferb/Modelfile
    gathering model components
    using existing layer sha256:aabd4debf0c8f08881923f2c25fc0fdeed24435271c2b3e92c4af36704040dbc
    using existing layer sha256:6e4c38e1172f42fdbff13edf9a7a017679fb82b0fde415a3e8b3c31c6ed4a4e4
    creating new layer sha256:7b88926a517b7e485e5b868e93690b091a6b5ea427cc02d472d68e36d5865cf2
    creating new layer sha256:0fb63f3d29b8521893106e4e631236d6ce977567fcab71c4c682c31d928fa427
    creating new layer sha256:0bf7528b44b3ef2ededf4ba6e2d7a069f08f09412424e9d9305d171b32359e8b
    writing manifest
    success
    root@ip-10-1-1-5:/root# docker exec ollama ollama create calvin-hobbes -f /root/.ollama/models/calvin-hobbes/Modelfile
    gathering model components
    using existing layer sha256:2af3b81862c6be03c769683af18efdadb2c33f60ff32ab6f83e42c043d6c7816
    creating new layer sha256:a662ea17df17520e8689f8f566ca6818aad41f745d4d392be2402b7018483b3b
    creating new layer sha256:1ffad0ca6061aa23819ab27af4fdbf58a185f2e8b680791a7327ca59b03135e4
    creating new layer sha256:7155a7e0f3d62b4bb4bfe745a3989767c3863ef6cdcfcedb47b808f389af2b32
    writing manifest
    success

#. Verify your new models are in the model list

.. code-block:: bash

    docker exec ollama ollama list

The output should resemble:

.. code-block:: bash

    root@ip-10-1-1-5:/root# docker exec ollama ollama list
    NAME                    ID              SIZE      MODIFIED
    calvin-hobbes:latest    f7a3b72a022f    637 MB    About a minute ago
    phineas-ferb:latest     93b0c5ef25d2    1.1 GB    2 minutes ago
    llama3.2:3b             a80c4f17acd5    2.0 GB    5 minutes ago
    deepseek-r1:1.5b        e0979632db5a    1.1 GB    5 minutes ago
    tinyllama:latest        2644915ede35    637 MB    6 minutes ago

#. Give your custom models a test run

First up, Calvin & Hobbes

.. code-block:: bash

    docker exec -it ollama ollama run calvin-hobbes
    >>> tell me about snowmen

Your responses will vary, LLMs are not digests regurgitating data. My response:

.. code-block:: bash

     I'm happy to help, but do you really want to hear about the most boring subject in all of
    Springfield?\nCalvin: bring it on! I've been practicing my snowman building skills and I
    just got some new ideas from reading one of Snoop Dogg's books\nHobbes: *sigh* Fine. So,
    there are several ways to build a snowman. The most basic is using three large snowballs,
    with the largest one at the bottom, and then stacking them on top of each other. You can use
    sticks for the arms and a carrot for the nose. And if you're feeling fancy, you can add some
    coal eyes and a hat.\nCalvin: that's it?! I didn't know they were so boring!\nHobbes: What
    do you mean? That's the basic design. There's also variations like using different shapes or
    colors, or even building a snowfort instead of a snowman.\nCalvin: but where's the challenge
    in that? I want to build a giant robot snowman with laser beams and a jetpack!\nHobbes:
    *shaking head* You're going to be disappointed. There isn't enough snow this year, and
    besides, Snoop Dogg's book was about cooking not building.\nCalvin: oh no! My idea is
    ruined! But I'll just have to build one anyway, and I know just what I can do to make it
    cooler than your boring old snowman!\nHobbes: *sigh* Just don't burn down the neighborhood
    with your reckless plans.

Next up, Phineas & Ferb

.. code-block:: bash

    docker exec -it ollama ollama run phineas-ferb
    >>> tell me about snowmen

And my response. Notice the tone and style is different than the calvin-hobbes model.

.. code-block:: bash

     I can do that.
    Phineas: \*\*excitedly\*\*That's the spirit! Okay, so we're going to build a snowman and
    have a snowball fight at the same time!\nFerb: *nods silently*

    Phineas: Alright, let's get started. We need to roll him into shape first.\nFerb: *starts
    rolling the snow on the ground\* (makes sounds of rolling)

    Phineas: \*\*laughs\*\* I know you are! Now, while Ferb is getting the snow ready, I'll go
    find some sticks and rocks for the arms and legs.

    (Phineas runs off to gather materials)

    (Ferb continues rolling the snow into a large ball)

    Phineas: *returns with sticks and rocks* Alright, it's time to put our snowman together!
    Ferb, can you help me with that?

    Ferb: *nods and starts building the arms and legs*

    Phineas: Great job, Ferb! Now we just need a face. Let's see... I know what we can use!

    (Phineas pulls out some coal from his pocket and begins to draw on the snowman)

    Ferb: (stays silent and continues to build)

    Phineas: There we go! Our snowman is complete!

    (The snowman stands tall, with Phineas proudly standing beside it)

    Phineas: Now that our snowman is done, let's have a snowball fight!

    (Ferb nods, and the two of them start throwing snowballs at each other)

    (Ferb is seen throwing a massive amount of snowballs in quick succession)

    Phineas: (laughs) Wow, Ferb! You're really showing off your skills now!

    (They continue to engage in an epic snowball battle, laughing and having fun together)

.. note::

    I tried at first to build these custom models based on tinyllama, but that model wasn't
    working with me at all!

**CHALLENGE** - Now create your own model file, load it into the model_data volume,
and create and run your custom model.

Recap
-----
In this lab, you:

- Customized a couple models based on llama3.2:3b and ran a simple test against it

This completes Module 1 working directly with models. Next we'll
take a look at client interfaces and frameworks.
