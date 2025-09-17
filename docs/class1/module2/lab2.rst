Lab 2.2 - Installing & Exploring the Fabric Framework
=====================================================

In this lab, we will look at a few use cases on how to use the Fabric framework to optimize your
workflows. Fabric doesn't have a native container, so we'll need to build one.

Install Fabric
--------------

In your deployment, click on the **Components** tab, and under **Systems**, click **Access** on the
Jumphost and select **WEB SHELL** as shown in the image below.

.. image:: images/00_jumphost_webshell_interface.png

1. Create a directory under the root user for the fabric Dockerfile.

.. code-block:: bash

    mkdir -p /root/fabric

2. Create the Dockerfile in /root/fabric.

.. code-block:: bash

    cat > /root/fabric/Dockerfile << 'EOF'
    # Dockerfile for Fabric Framework by Daniel Miessler
    # Simple build for Ubuntu with root user

    FROM golang:1.25-alpine

    # Install all required packages and tools
    RUN apk add --no-cache \
        git \
        ca-certificates \
        bash \
        curl \
        wget \
        vim \
        nano \
        jq \
        xclip \
        && rm -rf /var/cache/apk/*

    # Install Fabric
    RUN go install github.com/danielmiessler/fabric/cmd/fabric@latest

    # Set working directory
    WORKDIR /root

    # Create necessary directories
    RUN mkdir -p /root/.config/fabric

    # Set environment variables
    ENV PATH="/go/bin:$PATH"
    ENV FABRIC_CONFIG_DIR="/root/.config/fabric"

    # Create helpful aliases and setup info
    RUN echo 'alias pbpaste="xclip -selection clipboard -o"' >> /root/.bashrc && \
        echo 'alias pbcopy="xclip -selection clipboard"' >> /root/.bashrc && \
        echo '' >> /root/.bashrc && \
        echo '# Fabric Framework is ready!' >> /root/.bashrc && \
        echo '# First time setup: fabric --setup' >> /root/.bashrc && \
        echo '# List patterns: fabric --listpatterns' >> /root/.bashrc && \
        echo '# Usage: echo "text" | fabric --pattern summarize' >> /root/.bashrc && \
        echo '# Help: fabric -h' >> /root/.bashrc

    # Default command
    CMD ["/bin/bash"]
    EOF

3. Now change directory with the ``cd`` command into the /root/fabric directory and build the image.

.. code-block:: bash

    cd /root/fabric
    docker build -t fabric .

.. note::

    This will take several minutes.

4. After the build completes, **create and run a new container**. This
will dump you into the bash shell of the container.

.. code-block:: bash

    docker run -it --name fabric-shell -v fabric_config:/root/.config/fabric fabric

.. code-block:: bash

    f219740ed8c1:~# <------ The shell prompt has changed. You are now in the shell of the container>

Configure Fabric
----------------

1. Run setup

.. code-block:: bash

    fabric --setup

That should bring you to this text screen:

.. code-block:: bash

    Available plugins (please configure all required plugins)::

    AI Vendors [at least one, required]

            [1]     AIML
            [2]     Anthropic
            [3]     Azure
            [4]     Cerebras
            [5]     DeepSeek
            [6]     Exolab
            [7]     Gemini
            [8]     GrokAI
            [9]     Groq
            [10]    Langdock
            [11]    LiteLLM
            [12]    LM Studio
            [13]    Mistral
            [14]    Ollama
            [15]    OpenAI
            [16]    OpenRouter
            [17]    Perplexity
            [18]    SiliconCloud
            [19]    Together

    Tools

            [20]    Custom Patterns - Set directory for your custom patterns (optional)
            [21]    Default AI Vendor and Model [required]
            [22]    Jina AI Service - to grab a webpage as clean, LLM-friendly text (configured)
            [23]    Language - Default AI Vendor Output Language (configured)
            [24]    Patterns - Downloads patterns [required]
            [25]    Strategies - Downloads Prompting Strategies (like chain of thought) [required]
            [26]    YouTube - to grab video transcripts (via yt-dlp) and comments/metadata (via YouTube API)

    [Plugin Number] Enter the number of the plugin to setup (leave empty to skip):

We're going to focus on plugin 14 and Tools 21, 24, and 25.

2. Select plugin number 14 and hit **Enter**.


You should see this text output:

.. code-block:: bash

    [Ollama]

    Enter your Ollama URL (as a reminder, it is usually 'http://localhost:11434') (leave empty for 'http://localhost:11434' or type 'reset' to remove the value):

3. Enter ``http://10.1.1.5:11434`` and hit **Enter**, then skip the Ollama API Key step as we did not set one up.

4. Leave the default of ``20m`` for the HTTP timeout duration.

This should bring you back to the main text screen and you should see (configured) next to Ollama.

.. code-block:: bash

    Available plugins (please configure all required plugins)::

    AI Vendors [at least one, required]

            [1]     AIML
            [2]     Anthropic
            [3]     Azure
            [4]     Cerebras
            [5]     DeepSeek
            [6]     Exolab
            [7]     Gemini
            [8]     GrokAI
            [9]     Groq
            [10]    Langdock
            [11]    LiteLLM
            [12]    LM Studio
            [13]    Mistral
            [14]    Ollama (configured)
            [15]    OpenAI
            [16]    OpenRouter
            [17]    Perplexity
            [18]    SiliconCloud
            [19]    Together

    Tools

            [20]    Custom Patterns - Set directory for your custom patterns (optional)
            [21]    Default AI Vendor and Model [required]
            [22]    Jina AI Service - to grab a webpage as clean, LLM-friendly text (configured)
            [23]    Language - Default AI Vendor Output Language (configured)
            [24]    Patterns - Downloads patterns [required]
            [25]    Strategies - Downloads Prompting Strategies (like chain of thought) [required]
            [26]    YouTube - to grab video transcripts (via yt-dlp) and comments/metadata (via YouTube API)

    [Plugin Number] Enter the number of the plugin to setup (leave empty to skip):

5. Next, select 21 and hit **Enter**. For now, choose ``llama3.2:3b``. In my instance, that is number 3 but that
might be different for you. Skip the model context length.

.. code-block:: bash

    Available models:

    Ollama

            [1]     deepseek-r1:1.5b
            [2]     deepseek-r1:7b
            [3]     llama3.2:3b
            [4]     tinyllama:latest

    [Default]

    Enter the index the name of your default model (leave empty to skip):
    3

    Enter model context length (leave empty to skip):

6. This should bring you back to the main text screen again. Now select 24 for **Patterns**. Accept the defaults and
the Patterns should be cloned for you.

.. code-block:: bash

    Tools

            [20]    Custom Patterns - Set directory for your custom patterns (optional)
            [21]    Default AI Vendor and Model [required] (configured)
            [22]    Jina AI Service - to grab a webpage as clean, LLM-friendly text (configured)
            [23]    Language - Default AI Vendor Output Language (configured)
            [24]    Patterns - Downloads patterns [required]
            [25]    Strategies - Downloads Prompting Strategies (like chain of thought) [required]
            [26]    YouTube - to grab video transcripts (via yt-dlp) and comments/metadata (via YouTube API)

    [Plugin Number] Enter the number of the plugin to setup (leave empty to skip):
    24

    [Patterns Loader]

    Enter the default Git repository URL for the patterns (leave empty for 'https://github.com/danielmiessler/fabric.git' or type 'reset' to remove the value):


    Enter the default folder in the Git repository where patterns are stored (leave empty for 'data/patterns' or type 'reset' to remove the value):

    Downloading patterns and Populating /home/fabricuser/.config/fabric/patterns...

    Cloning repository https://github.com/danielmiessler/fabric.git (path: data/patterns)...
    Downloaded 225 patterns to temporary directory
    ✅ Successfully downloaded and installed patterns to /home/fabricuser/.config/fabric/patterns
    📝 Created unique patterns file with 225 pattern

7. Back at the main screen, select **Strategies** at 25 and Accept the defaults.

.. note:: I had to do this twice for it to show configured.

.. code-block:: bash

    [Prompt Strategies]

    Enter the default Git repository URL for the strategies (leave empty for 'https://github.com/danielmiessler/fabric.git' or type 'reset' to remove the value):


    Enter the default folder in the Git repository where strategies are stored (leave empty for 'data/strategies' or type 'reset' to remove the value):

    Downloading strategies and Populating /home/fabricuser/.config/fabric/strategies...


    Available plugins (please configure all required plugins)::

    AI Vendors [at least one, required]

            [1]     AIML
            [2]     Anthropic
            [3]     Azure
            [4]     Cerebras
            [5]     DeepSeek
            [6]     Exolab
            [7]     Gemini
            [8]     GrokAI
            [9]     Groq
            [10]    Langdock
            [11]    LiteLLM
            [12]    LM Studio
            [13]    Mistral
            [14]    Ollama (configured)
            [15]    OpenAI
            [16]    OpenRouter
            [17]    Perplexity
            [18]    SiliconCloud
            [19]    Together

    Tools

            [20]    Custom Patterns - Set directory for your custom patterns (optional)
            [21]    Default AI Vendor and Model [required] (configured)
            [22]    Jina AI Service - to grab a webpage as clean, LLM-friendly text (configured)
            [23]    Language - Default AI Vendor Output Language (configured)
            [24]    Patterns - Downloads patterns [required] (configured)
            [25]    Strategies - Downloads Prompting Strategies (like chain of thought) [required] (configured)
            [26]    YouTube - to grab video transcripts (via yt-dlp) and comments/metadata (via YouTube API)

    [Plugin Number] Enter the number of the plugin to setup (leave empty to skip):

8. Hit **Enter** with no plugin number to Exit setup. Fabric is now configured.

Fabric Patterns
---------------

Patterns are reusable, structured prompts designed to accomplish specific tasks through large
language models like Claude or GPT. These patterns serve as templates that standardize how to
approach common problems, from analyzing data and extracting insights to summarizing content and
generating reports. The Fabric framework, created by Daniel Miessler, provides a collection of
these patterns that can be applied consistently across different contexts, helping users achieve
more reliable and focused outputs from AI systems. Each pattern typically includes specific
instructions, formatting requirements, and examples that guide the AI toward producing the desired result.

You can use the `extract_wisdom
<https://github.com/danielmiessler/Fabric/blob/main/data/patterns/extract_wisdom/system.md>`_ pattern by
piping text to fabric. (You can do this directly with the yt-dlp command if installed to automatically
grab transcripts from YouTube, but that requires an API key.) Just copy the contents of `this youtube
transcript <../resources/yt-transcript.html>`_ into your fabric container (I named mine yt-transcript.txt)
and then pipe that to fabric like this:

.. code-block:: bash

    echo yt-transcript.txt | fabric -sp extract_wisdom --model llama3.2:3b

I got the following output against the llama3.2:3b model in my environment (it took a few minutes). Your output should
resemble the following, but won't match exactly.

.. code-block:: bash

        # SUMARY
        Insights from a text content related to identity, purpose, human flourishing, technology, and artificial intelligence by an unknown speaker.

        # IDEAS
        • AI can enhance human creativity but may also lead to a loss of human touch in art.
        • Embracing imperfection can be more beneficial than striving for perfection in personal growth.
        • The future of humanity depends on balancing individuality with collective progress.
        • Reading widely is essential for expanding perspectives and fostering empathy.
        • The best way to learn new skills is through hands-on experience and experimentation.
        • Continuous improvement is key to personal growth and overcoming challenges.
        • Technology can be both empowering and isolating, depending on how it's used.
        • Memes can be a powerful tool for spreading ideas and promoting social change.
        • Learning from failure is crucial for success in various aspects of life.
        • Self-reflection and introspection are essential tools for personal growth and self-awareness.
        • The pursuit of purpose is a lifelong journey, not a destination.
        • Human flourishing requires balance, resilience, and adaptability.
        • Artificial intelligence has the potential to revolutionize industries and improve lives.
        • Over-reliance on technology can lead to decreased creativity and innovation.
        • Personal habits such as regular exercise and healthy eating are crucial for well-being.
        • The importance of community and social connections cannot be overstated in personal growth.
        • Embracing lifelong learning is essential for staying relevant in a rapidly changing world.
        • Creativity and imagination are essential components of human flourishing.

        # INSIGHTS
        • Embracing imperfection can lead to increased self-acceptance and reduced anxiety.
        • The pursuit of purpose requires embracing uncertainty and taking calculated risks.
        • Technology can be both a means to an end and an end in itself, depending on how it's used.
        • Human flourishing is often the result of finding balance between competing desires and values.
        • Self-reflection is essential for personal growth but should not be overemphasized at the expense of action.
        • The best way to improve oneself is through consistent effort and self-awareness.
        • Creativity can be cultivated through practice, patience, and persistence.
        • Personal growth requires embracing challenges and learning from failures.

        # QUOTES
        • "The future belongs to those who believe in the beauty of their dreams." - Eleanor Roosevelt
        • "You are never too old to set another goal or to dream a new dream." - C.S. Lewis
        • "The only way to do great work is to love what you do." - Steve Jobs

        # HABITS
        • Regularly setting aside time for personal growth and self-reflection.
        • Engaging in physical activity, such as exercise or sports, to improve mental health.
        • Prioritizing sleep schedule to ensure adequate rest and recovery.
        • Practicing mindfulness through meditation or deep breathing exercises.
        • Reading widely to expand perspectives and foster empathy.

        # FACTS
        • The world's population is projected to reach 9.7 billion by 2050.
        • The average person spends around 4 hours and 20 minutes per day using technology.
        • Artificial intelligence has the potential to revolutionize industries such as healthcare and finance.
        • The human brain contains over 100 billion neurons and trillions of connections.
        • The world's largest living organism is a fungus that covers over 2,200 acres.

        # REFERENCES
        • Books: "The 7 Habits of Highly Effective People" by Stephen Covey
        • Writing tool: Medium
        • Artistic inspiration: Impressionist art movement
        • Project idea: Starting a blog to share personal growth experiences

        # ONE-SENTENCE TAKEAWAY
        Embracing imperfection and uncertainty is essential for finding purpose and achieving human flourishing in a rapidly changing world.

        # RECOMMENDATIONS
        • Regularly set aside time for self-reflection and personal growth.
        • Prioritize physical activity and exercise for improved mental health.
        • Read widely to expand perspectives and foster empathy.
        • Practice mindfulness through meditation or deep breathing exercises.

That's pretty powerful, right? In short order, you can get an idea on whether or not a video is
worth your time, particularly so if you also use the rate_content pattern. But again, this is not limited
to videos, you can use this and other patterns against any text source.

Bonus - Personal Growth Tracking with Telos
-------------------------------------------

Telos is a Greek word generally meaning goal, purpose, fulfillment. Daniel Miessler created a
`telos framework <https://github.com/danielmiessler/Telos>`_ for helping people realize this.

I created `a mock TELOS <../resources/telos.html>`_ file for the persona of a network
engineer who is struggling adapting to modern tools and is facing anxiety on being left behind. There
are several telos-based patterns that you can explore, but in this example, I used the `t_red_team_thinking
pattern <https://github.com/danielmiessler/Fabric/blob/main/data/patterns/t_red_team_thinking/system.md>`_.

This one took several minutes to run, so be patient! (I copied the mock telos from above into a file in the fabric
container.)

.. code-block:: bash

    cat telos | fabric -sp t_red_team_thinking --model deepseek-r1:7b

The output should resemble this:

.. code-block:: bash

    <think>
    Okay, so I need to help this person by analyzing their situation and providing some recommendations based on the TELOS file they provided. Let me try to break it down step by step.

    First, looking at the problems: P1 is about traditional network engineers being left behind by rapid cloud and AI transformation. That makes sense because as companies move to the cloud and adopt AI technologies, their network expertise needs are changing too. So, there's a gap between what they're learning now and what's required in the future.

    Problem P2 talks about management creating anxiety through pressure for technical growth. I can see how that might happen—when people feel like they have to keep up with new stuff all the time, it can cause stress. It's important to create environments where engineers aren't just trying to grow but actually enjoy learning and don't feel overwhelmed.

    Problem P3 is about technology fragmentation making it hard for engineers to choose relevant skills. With so many tools and technologies out there, it's tough to know which ones are worth investing time in. This can lead to confusion and wasted effort on things that aren't even necessary anymore.

    Now looking at the missions: M1 aims to bridge traditional networking with modern cloud-native and AI-driven infrastructure. That seems like a good start because it directly addresses P1 by integrating their existing skills into newer technologies. M2 is about creating sustainable learning cultures without causing anxiety, which ties into P2. And M3 simplifies skill progression, helping tackle P3.

    The narratives give different ways to understand the mission, so that's helpful for communication. The goals are specific and have clear metrics, which is great for tracking progress. Challenges like C1 and C2 are common in this situation—imposter syndrome and time constraints. Strategies S1 and S2 seem tailored to address these challenges.

    The projects like PJT1 and PJT2 are practical steps toward the missions. They involve teaching networking fundamentals while learning new tools, which should help bridge the gap. The history shows a progression of skills over time, starting from basic automation with Python to more complex tasks involving Kubernetes and microservices.

    Looking at the log entries, it seems like the person is making steady progress but also facing setbacks. For example, they struggled with some networking concepts initially but found success after persistence. They also had moments where imposter syndrome kicked in, especially when dealing with younger engineers who seemed to grasp things faster.

    From this analysis, I can see that the key areas of focus should be on creating a structured learning path (as per M3), fostering mentorship and peer learning (M2), and bridging traditional skills with modern technologies (M1). The strategies like teaching fundamentals alongside new tools and gradually integrating new concepts into existing scripts seem effective.

    I also notice that the person is actively trying to build projects, which is good for applying what they learn. However, time management issues might be a challenge, especially as they juggle work and personal life. Creating a sustainable culture around learning without burnout will require thoughtful approach—maybe through scheduled learning times, peer support, and clear milestones.

    In terms of recommendations, I should focus on supporting consistent practice, creating safe spaces for experimentation without fear of failure, and gradually building complexity in their projects to avoid overwhelming them. Also, leveraging mentorship and peer learning can help distribute the pressure and make the transition smoother.
    </think>

    To address the challenges faced by the individual, here are strategic recommendations focusing on structured learning, fostering a positive environment, and gradual skill development:

    1. **Structured Learning Path**: Implement a clear progression of skills that bridge traditional networking with modern technologies. This should include courses like Kubernetes networking and microservices communication, supported by practical projects.
    2. **Mentorship and Peer Learning**: Establish a mentorship program to pair experienced engineers with those still learning, creating a supportive environment for knowledge exchange. Encourage peer-to-peer learning through lunch-and-learn sessions and collaborative projects.
    3. **Sustainable Learning Culture**: Create a work environment that values continuous learning without causing anxiety. This can be achieved by setting realistic goals, providing resources, and recognizing progress to maintain motivation.
    4. **Gradual Skill Integration**: Introduce new concepts into existing scripts incrementally to avoid overwhelming the learning process. This approach allows for gradual understanding and reduces the risk of burnout.
    5. **Time Management Strategies**: Implement time management techniques such as scheduled learning blocks and flexible work arrangements to accommodate personal responsibilities without compromising professional growth.

    By focusing on these areas, the individual can effectively navigate the transition to modern technologies while maintaining a healthy balance between professional and personal life.

There are so many frameworks for hacking yourself these days and the ability to have your own AI coach/therapist
is pretty intriguing. I'm still exploring all the patterns, and you can create your own as well.

Recap
-----
You now have the following:

- A powerful command-line AI Assistant to optimize your data ingestion and note taking experiences.


This completes Module 2. Click Next to move on to Module 3.








