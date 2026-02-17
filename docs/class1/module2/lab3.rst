Lab 2.3 - Installing & Exploring the Fabric Framework
=====================================================

In this lab, we will look at a few use cases on how to use the Fabric framework to optimize your
workflows. Fabric doesn't have a native container, so we'll need to build one.

Install Fabric
--------------

In your deployment, click on the **Components** tab, and under **Systems**, click **Access** on the
Jumphost and select **WEB SHELL** as shown in the image below.

.. image:: images/00_appserver_webshell_interface.png

1. Change directory into fabric and review the compose.yaml and Dockerfile, then run the container.

.. code-block:: console

    cd /root/fabric
    docker compose run --rm fabric

If doing for the first time it takes a few minutes and resemble the output below. In our case since we've
pre-built the image and container, you'll just be dumped into the container's shell prompt (last line of
this output.)

.. code-block:: console

    root@ip-10-1-1-4:/root/fabric# docker compose run --rm fabric
    [+] Creating 2/2
     ✔ Network fabric_default         Created                                                                                                                                             0.1s
     ✔ Volume "fabric_fabric_config"  Created                                                                                                                                             0.0s
    [+] Building 179.7s (12/12) FINISHED
     => [internal] load local bake definitions                                                                                                                                            0.0s
     => => reading from stdin 326B                                                                                                                                                        0.0s
     => [internal] load build definition from Dockerfile                                                                                                                                  0.0s
     => => transferring dockerfile: 1.16kB                                                                                                                                                0.0s
     => [internal] load metadata for docker.io/library/golang:1.25-alpine                                                                                                                 1.0s
     => [internal] load .dockerignore                                                                                                                                                     0.0s
     => => transferring context: 2B                                                                                                                                                       0.0s
     => [1/6] FROM docker.io/library/golang:1.25-alpine@sha256:aee43c3ccbf24fdffb7295693b6e33b21e01baec1b2a55acc351fde345e9ec34                                                           8.3s
     => => resolve docker.io/library/golang:1.25-alpine@sha256:aee43c3ccbf24fdffb7295693b6e33b21e01baec1b2a55acc351fde345e9ec34                                                           0.0s
     => => sha256:c3dc5d5e8cf34ccb2172fb8d1aa399aa13cd8b60d27bba891d18e3b436a0c5f6 1.92kB / 1.92kB                                                                                        0.0s
     => => sha256:eddaa94abd7ba615b9d760e7e70d75af2567a9332a3c959bb79a64a2f74bf682 2.08kB / 2.08kB                                                                                        0.0s
     => => sha256:2d35ebdb57d9971fea0cac1582aa78935adf8058b2cc32db163c98822e5dfa1b 3.80MB / 3.80MB                                                                                        0.2s
     => => sha256:85e8836fcdb2966cd3e43a5440ccddffd1828d2d186a49fa7c17b605db8b3bb3 291.15kB / 291.15kB                                                                                    0.3s
     => => sha256:91631faa732ae651543f888b70295cbfe29a433d3c8da02b9966f67f238d3603 60.15MB / 60.15MB                                                                                      1.5s
     => => sha256:aee43c3ccbf24fdffb7295693b6e33b21e01baec1b2a55acc351fde345e9ec34 10.29kB / 10.29kB                                                                                      0.0s
     => => extracting sha256:2d35ebdb57d9971fea0cac1582aa78935adf8058b2cc32db163c98822e5dfa1b                                                                                             0.2s
     => => sha256:f3f5ae8826faeb0e0415f8f29afbc9550ae5d655f3982b2924949c93d5efd5c8 126B / 126B                                                                                            0.3s
     => => sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1 32B / 32B                                                                                              0.4s
     => => extracting sha256:85e8836fcdb2966cd3e43a5440ccddffd1828d2d186a49fa7c17b605db8b3bb3                                                                                             0.1s
     => => extracting sha256:91631faa732ae651543f888b70295cbfe29a433d3c8da02b9966f67f238d3603                                                                                             6.3s
     => => extracting sha256:f3f5ae8826faeb0e0415f8f29afbc9550ae5d655f3982b2924949c93d5efd5c8                                                                                             0.0s
     => => extracting sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1                                                                                             0.0s
     => [2/6] RUN apk add --no-cache     git     ca-certificates     bash     curl     wget     vim     nano     jq     xclip     && rm -rf /var/cache/apk/*                              3.7s
     => [3/6] RUN go install github.com/danielmiessler/fabric/cmd/fabric@latest                                                                                                         158.9s
     => [4/6] WORKDIR /root                                                                                                                                                               0.0s
     => [5/6] RUN mkdir -p /root/.config/fabric                                                                                                                                           0.2s
     => [6/6] RUN echo 'alias pbpaste="xclip -selection clipboard -o"' >> /root/.bashrc &&     echo 'alias pbcopy="xclip -selection clipboard"' >> /root/.bashrc &&     echo '' >> /root  0.3s
     => exporting to image                                                                                                                                                                7.0s
     => => exporting layers                                                                                                                                                               7.0s
     => => writing image sha256:d37a22ea43df675a87aa0c3260ea6f9aa3687c07238f75901e6443cfb84b531f                                                                                          0.0s
     => => naming to docker.io/library/fabric-fabric                                                                                                                                      0.0s
     => resolving provenance for metadata file                                                                                                                                            0.0s
    b37603eed0c1:~#

You'll notice you are now at the container shell prompt instead of at the App Server prompt.

Configure Fabric
----------------

1. Run setup

.. code-block:: console

    fabric --setup

That should resemble the following output:

.. code-block:: console

    b1b224ae5f77:~# fabric --setup

    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    🎉 Welcome to Fabric! Let's get you set up
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

    📥 Step 1: Downloading patterns (required for Fabric to work)..

    [Patterns Loader]

    Enter the default Git repository URL for the patterns (leave empty for 'https://github.com/danielmiessler/fabric.git' or type 'reset' to remove the value):

Accept the default and press enter. This should then present the following:

.. code-block:: console

    Enter the default folder in the Git repository where patterns are stored (leave empty for 'data/patterns' or type 'reset' to remove the value):

Accept the default again for the patterns and press enter. This should present the following:

.. code-block:: console

    Downloading patterns and Populating /root/.config/fabric/patterns...


    Cloning repository https://github.com/danielmiessler/fabric.git (path: data/patterns)...
    Downloaded 241 patterns to temporary directory
    ✅ Successfully downloaded and installed patterns to /root/.config/fabric/patterns
    📝 Created unique patterns file with 241 pattern

    📥 Step 2: Downloading strategies (required for Fabric to work)..

    [Prompt Strategies]

    Enter the default Git repository URL for the strategies (leave empty for 'https://github.com/danielmiessler/fabric.git' or type 'reset' to remove the value):

Accept the default again for the strategies and the repo and press enter. This should present the following:

.. code-block:: console

    Enter the default folder in the Git repository where strategies are stored (leave empty for 'data/strategies' or type 'reset' to remove the value):
    Downloading strategies and Populating /root/.config/fabric/strategies...


    Cloning repository https://github.com/danielmiessler/fabric.git (path: data/strategies)...
    Downloaded 9 strategies
    ✅ Successfully downloaded and installed strategies to /root/.config/fabric/strategies

    🤖 Step 3: Configure an AI provide
       Fabric needs at least one AI provider to work.
       You'll be able to add more providers later with 'fabric --setup'


    Available AI Providers::



            [1]     Abacus
            [2]     AIML
            [3]     Anthropic
            [4]     Azure
            [5]     Cerebras
            [6]     Copilot
            [7]     DeepSeek
            [8]     DigitalOcean
            [9]     Exolab
            [10]    Gemini
            [11]    GitHub
            [12]    GrokAI
            [13]    Groq
            [14]    Infermatic
            [15]    Langdock
            [16]    LiteLLM
            [17]    LM Studio
            [18]    Mammouth
            [19]    MiniMax
            [20]    Mistral
            [21]    Novita AI
            [22]    Ollama
            [23]    OpenAI
            [24]    OpenRouter
            [25]    Perplexity
            [26]    SiliconCloud
            [27]    Together
            [28]    Venice AI
            [29]    VertexAI
            [30]    Z AI

    [AI Provider Number] Enter the number of the AI provider to configure (leave empty to skip):

Choose the Ollama option (22 here, but might be different for you) and hit enter. Make sure to enter http://10.1.1.5:11435 and then skip the ollama key and the timeout.

.. code-block: console

    [Ollama]

    Enter your Ollama URL (as a reminder, it is usually http://localhost:11434') (leave empty for 'http://localhost:11434' or type 'reset' to remove the value):http://10.1.1.5:11435

    Enter your Ollama API KEY (leave empty to skip):

    Specify HTTP timeout duration for Ollama requests (e.g. 30s, 5m, 1h) (leave empty for '20m' or type 'reset' to remove the value):

Now choose the qwen2.5:7b-instruct-q5_0 model for this, which is number 7 for me.

.. code-block:: console

    ⚙  Step 4: Setting default vendor and model...

    Available models:

    Ollama

            [1]     calvin-hobbes:latest
            [2]     code-review:latest
            [3]     codellama:latest
            [4]     deepseek-r1:1.5b
            [5]     deepseek-r1:7b
            [6]     llama3.2:3b
            [7]     qwen2.5:7b-instruct-q5_0
            [8]     tinyllama:latest

    [Default]

    Enter the index or the name of your default model (leave empty to skip):

Finally, skip the context length.

.. code-block:: console

    Enter model context length (leave empty to skip):

    DEFAULT_VENDOR: Ollama
    DEFAULT_MODEL: qwen2.5:7b-instruct-q5_0

    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    ✅ Setup complete! You can now use Fabric.
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

    Next steps:
      • List available patterns: fabric -l
      • Try a pattern: echo 'your text' | fabric --pattern summarize
      • Configure more settings: fabric --setup


    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    Configuration Status:
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
      ✓ AI Provider configured
      ✓ Default vendor/model set: Ollama/qwen2.5:7b-instruct-q5_0
      ✓ Patterns downloaded
      ✓ Strategies downloaded
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

    ✓ All required components configured!

Fabric is now configured!

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
grab transcripts from YouTube, but that requires an API key.) The contents of `this youtube
transcript <../resources/yt-transcript.html>`_ are already in your fabric container (named yt-transcript.txt).
Run the following command:

.. code-block:: console

    fabric -sp extract_wisdom < yt-transcript.txt

.. note::

    We set the default model to qwen2.5:7b-instruct-q5_0, but if you want to test other models you can by appending **--model
    <model name>** to the fabric command shown above. Smaller models with tight context windows don't fair well
    with the fabric patterns, however, because the system prompts are quite large.

I got the following output against the qwen2.5:7b-instruct-q5_0 model in my environment (it took a few minutes). Your output should
resemble the following, but won't match exactly.

.. code-block:: console

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

This one took several minutes to run, so be patient! (The file is already in the fabric
container.). Run the following command:

.. code-block:: console

    fabric -sp t_red_team_thinking < telos.txt

The output should resemble this:

.. code-block:: console

    def82b36d3d6:~# fabric -sp t_red_team_thinking < telos.txt
    - Red-teaming identified that the Telos lacks a clear roadmap for addressing imposter syndrome and limited time for learning, despite these being significant challenges.
    - The strategies outlined are fragmented and need consolidation; focusing on mutual mentorship while integrating cloud-native technologies could streamline progress.
    - The narrative and missions do not directly address the anxiety management aspect, which is crucial for maintaining motivation and reducing burnout.
    - The goals lack specific timelines and performance metrics for certain tasks, such as the Kubernetes networking course and service mesh evaluation.
    - The log entries show a mix of successes and frustrations but lack structured reflection on how to balance traditional and modern skills effectively.

    ### Recommendations

    1. Develop a detailed plan for addressing imposter syndrome through regular peer support sessions and clear communication with management about skill gaps.
    2. Consolidate strategies into a cohesive roadmap that integrates mutual mentorship, gradual learning in production, and targeted training programs.
    3. Enhance the narrative to emphasize how balanced skill development reduces anxiety and fosters a supportive environment for growth.
    4. Add specific timelines and performance metrics to goals, such as completing the Kubernetes networking course by Q1 2026 and achieving a 90% success rate in service mesh evaluations.
    5. Include structured reflections on each day's log entries to identify patterns and actionable steps for overcoming learning challenges.

There are so many frameworks for hacking yourself these days and the ability to have your own AI coach/therapist
is pretty intriguing. I'm still exploring all the patterns, and you can create your own as well.

Recap
-----
You now have the following:

- A powerful command-line AI Assistant to optimize your data ingestion and note taking experiences.

In the **web shell** exit the container by typing **exit**


This completes Module 2. Click Next to move on to Module 3.








