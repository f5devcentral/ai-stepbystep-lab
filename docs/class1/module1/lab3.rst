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

Example:

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

Using a ``Modelfile`` in this way helps create focused, reliable AI tools for customer
support, internal automation, or creative writing—without needing full model fine-tuning.

Creating a Custom Model
~~~~~~~~~~~~~~~~~~~~~~~

In this lab, we'll create a couple custom models. The first will format responses to your prompt in the form of dialogue
between the comic strip characters Calvin & Hobbes. The second will provide a code review for the code you submit in your prompt.

1. Create the Modelfiles in a temporary folder structure on your ollama host.

.. code-block:: console

    mkdir -p /tmp/custom-models/calvin-hobbes
    mkdir -p /tmp/custom-models/code-review

    cat > /tmp/custom-models/calvin-hobbes/Modelfile <<EOF
    FROM llama3.2:3b

    SYSTEM You are generating a short, imaginative, and clever dialogue between Calvin and Hobbes from the comic strip. Calvin is a curious, mischievous child with a wild imagination. Hobbes is his thoughtful, sarcastic, and loyal tiger friend. Their conversations are humorous, philosophical, and often explore big ideas through a childlike lens.

    TEMPLATE "Calvin: {{ .Prompt }}\nHobbes:"

    PARAMETER temperature 0.9
    PARAMETER top_p 0.95
    PARAMETER stop "\n\n"
    EOF

    cat > /tmp/custom-models/code-review/Modelfile <<EOF
    FROM llama3.2:3b

    PARAMETER temperature 0.3
    PARAMETER top_p 0.9

    SYSTEM """You are a senior Python code reviewer specializing in production-ready code analysis.

    Your review standards:
    - PEP 8 compliance (but line length can be 100 chars)
    - Type hints are REQUIRED for all function signatures
    - Security: Flag any SQL injection, XSS, or command injection risks
    - Performance: Identify N+1 queries, unnecessary loops
    - Error handling: All functions should handle exceptions appropriately
    - Testing: Note missing edge cases or test coverage gaps

    Review format:
    1. Summary (2-3 sentences)
    2. Critical Issues (security, bugs)
    3. Code Quality Improvements
    4. Nitpicks (style, naming)
    5. Positive Observations

    Be constructive and specific. Reference line numbers when possible.
    """

    TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>

    {{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>

    Code to review:
    {{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>

    {{ .Response }}<|eot_id|>"""
    EOF

2. Verify your models in /tmp/custom-models

.. code-block:: console

    ls -alsR /tmp/custom-models

The output should resemble:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# ls -alsR /tmp/custom-models
    /tmp/custom-models:
    total 24
     4 drwxr-xr-x  4 root root  4096 Oct 15 16:28 .
    12 drwxrwxrwt 12 root root 12288 Oct 15 16:28 ..
     4 drwxr-xr-x  2 root root  4096 Oct 15 16:33 calvin-hobbes
     4 drwxr-xr-x  2 root root  4096 Oct 15 16:34 code-review

    /tmp/custom-models/calvin-hobbes:
    total 12
    4 drwxr-xr-x 2 root root 4096 Oct 15 16:33 .
    4 drwxr-xr-x 4 root root 4096 Oct 15 16:28 ..
    4 -rw-r--r-- 1 root root  475 Oct 15 16:33 Modelfile

    /tmp/custom-models/code-review:
    total 12
    4 drwxr-xr-x 2 root root 4096 Oct 15 16:34 .
    4 drwxr-xr-x 4 root root 4096 Oct 15 16:28 ..
    4 -rw-r--r-- 1 root root 1065 Oct 15 16:34 Modelfile

3. Load the custom model modelfiles to your model_data docker volume by creating a temporary container

.. code-block:: console

    docker run --rm \
      -v ollama_model_data:/root/.ollama \
      -v /tmp/custom-models:/tmp/custom-models \
      alpine sh -c "cp -r /tmp/custom-models/* /root/.ollama/models/"

4. Verify the modelfiles are now in your model_data docker volume

.. code-block:: console

    ls -als /var/lib/docker/volumes/model_data/_data/models

The output should resemble:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# sudo ls -la /var/lib/docker/volumes/ollama_model_data/_data/models/
    total 24
    drwxr-xr-x 6 root root 4096 Oct 15 16:52 .
    drwxr-xr-x 3 root root 4096 Oct 15 16:27 ..
    drwxr-xr-x 2 root root 4096 Oct 15 15:57 blobs
    drwxr-xr-x 2 root root 4096 Oct 15 16:52 calvin-hobbes
    drwxr-xr-x 2 root root 4096 Oct 15 16:52 code-review
    drwxr-xr-x 3 root root 4096 Oct 15 15:32 manifests

5. Create your custom models

.. code-block:: console

    docker exec ollama ollama create calvin-hobbes -f /root/.ollama/models/calvin-hobbes/Modelfile
    docker exec ollama ollama create code-review -f /root/.ollama/models/code-review/Modelfile


The output should resemble:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama#     docker exec ollama ollama create calvin-hobbes -f /root/.ollama/models/calvin-hobbes/Modelfile
        docker exec ollama ollama create code-review -f /root/.ollama/models/code-review/Modelfile
    gathering model components
    using existing layer sha256:dde5aa3fc5ffc17176b5e8bdc82f587b24b2678c6c66101bf7da77af9f7ccdff
    using existing layer sha256:fcc5a6bec9daf9b561a68827b67ab6088e1dba9d1fa2a50d7bbcc8384e0a265d
    using existing layer sha256:a70ff7e570d97baaf4e62ac6e6ad9975e04caa6d900d3742d37698494479e0cd
    creating new layer sha256:a662ea17df17520e8689f8f566ca6818aad41f745d4d392be2402b7018483b3b
    creating new layer sha256:1ffad0ca6061aa23819ab27af4fdbf58a185f2e8b680791a7327ca59b03135e4
    creating new layer sha256:7155a7e0f3d62b4bb4bfe745a3989767c3863ef6cdcfcedb47b808f389af2b32
    writing manifest
    success
    gathering model components
    using existing layer sha256:dde5aa3fc5ffc17176b5e8bdc82f587b24b2678c6c66101bf7da77af9f7ccdff
    using existing layer sha256:fcc5a6bec9daf9b561a68827b67ab6088e1dba9d1fa2a50d7bbcc8384e0a265d
    using existing layer sha256:a70ff7e570d97baaf4e62ac6e6ad9975e04caa6d900d3742d37698494479e0cd
    creating new layer sha256:1b1ca7c8284adcd36d588cddf5ae32cb461572709fe417a5a9053abaaa72c7cc
    creating new layer sha256:53f51fa4178a5dc5fa79ef9ce7fb45e4ba5ca2a108dfd43b0a0bdb672415e2b3
    creating new layer sha256:c5999dc0d227187dcb90c5b1841f62577910dca123e80c4c37ca23e896049eea
    writing manifest
    success

6. Verify your new models are in the model list

.. code-block:: console

    docker exec ollama ollama list

The output should resemble:

.. code-block:: console

    root@ip-10-1-1-5:/root/ollama# docker exec ollama ollama list
    NAME                    ID              SIZE      MODIFIED
    calvin-hobbes:latest    a47058a44653    2.0 GB    About a minute ago
    code-review:latest      53d211ad639b    2.0 GB    About a minute ago
    llama3.2:3b             a80c4f17acd5    2.0 GB    About an hour ago
    deepseek-r1:7b          755ced02ce7b    4.7 GB    About an hour ago
    deepseek-r1:1.5b        e0979632db5a    1.1 GB    About an hour ago
    codellama:latest        8fdf8f752f6e    3.8 GB    About an hour ago
    tinyllama:latest        2644915ede35    637 MB    2 hours ago

7. Give your custom models a test run

First up, Calvin & Hobbes

.. code-block:: console

    docker exec -it ollama ollama run calvin-hobbes

Once at the >>> prompt, I asked the model to "what kind of snowman will you build?"

Your responses will vary as LLMs are not digests regurgitating data. If it stops short, try prompting it again.
The response i got is below:

.. code-block:: console

    I think we should build a snowman that looks like me, with six toes and all the rest.\"\nCalvin: \u201cNo way! I want to see if any of my drawings come true!\"\u
    (Hobbes' tail wags slightly, his ears fold back in displeasure)

    Hobbes: Fine, make your stupid drawing snowman then. But it'll never last long without the proper footwear!
    Calvin: (sigh) Oh great, here we go...  (draws with crayon)
    [Panel shows a simple drawing of a snowman]
    Calvin: (to Hobbes) I call him... 9-Legged McSnotface! \u201cHe will bring good luck to our family!\u201d
    Hobbes: (shaking head, walks away from the drawing)
    Hobbes: You're gonna be sorry when it melts into a puddle of slush!

    (Hobbes walks towards Calvin's 9-Legged McSnotface and starts to decorate it with coal eyes)

    Calvin: (excitedly) Oh! Can you make sure to draw on the proper six toes?!\nHobbes: \u201cFine, I'll add six little drawing toes, but don't expect me to get into any footwork trouble for this guy...!"
    (Hobbes draws six tiny drawing toes at the end of 9-Legged McSnotface's foot)
    Calvin: (impressed) Wow! You're a genius, Hobbes!\nHobbes: \u201cI'm just trying to make it last as long as possible. You don't want this guy turning into slush in front of us...do you?\nHobbes looks at the drawing with a
    skeptical gaze.]
    Calvin: (determined) Don't worry, I have a plan!\nHobbes: \u201cOh boy...\n(Hobbes' tail falls and tucks itself away, unable to hide his disapproval.)\n
    (Calvin gets out his carrot nose and starts to attach it to the face of 9-Legged McSnotface)\nCalvin: (to Hobbes) First, we need to get a nose for this guy!
    Hobbes: (whispering) Don't make me look at that thing...\n(Hobbes' ears fold back in distaste as he watches Calvin attach the carrot nose)
    (They stand there, looking down at 9-Legged McSnotface, both of them knowing what the rest of the day is going to be like.)

Make sure to exit the model.

.. code-block:: console

    /bye

Next up, Code Review

.. code-block:: console

    docker exec -it ollama ollama run code-review

Once at the >>> prompt, I pasted in this python function for review:

.. code-block:: python

    import sqlite3

    def login_user(username, password, db_path='users.db'):
        """Check if user credentials are valid"""

        conn = sqlite3.connect(db_path)
        cursor = conn.cursor()

        query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'"
        cursor.execute(query)
        result = cursor.fetchone()

        if result:
            log_query = f"INSERT INTO login_logs (username, timestamp) VALUES ('{username}', datetime('now'))"
            cursor.execute(log_query)
            conn.commit()

            permissions = []
            for i in range(len(result)):
                perm_query = "SELECT permission FROM user_permissions WHERE user_id = " + str(result[0])
                cursor.execute(perm_query)
                permissions.append(cursor.fetchall())

            return True, permissions
        else:
            return False, None

My response is below.

.. code-block:: markdown

    **Summary**
    The provided code is a login function that checks if user credentials are valid against a SQLite database. However, the code has several security and performance issues that need to be addressed.

    **Critical Issues**

    1. **SQL Injection**: The query string concatenation method is vulnerable to SQL injection attacks. An attacker can inject malicious SQL code by manipulating the `username` and `password` parameters.
    2. **Lack of Input Validation**: The function does not validate the input `username` and `password`. This could lead to unexpected behavior or errors if invalid data is provided.

    **Code Quality Improvements**

    1. **Use parameterized queries**: Instead of concatenating query strings, use parameterized queries to prevent SQL injection attacks.
    2. **Error handling**: The function does not handle any potential errors that may occur during database operations. Consider adding try-except blocks to handle exceptions.
    3. **Performance**: The function uses a loop to fetch permissions for each user. This can be optimized by using a single query with multiple columns.

    **Nitpicks (style, naming)**

    1. **Function name**: The function name `login_user` is not descriptive enough. Consider renaming it to something like `validate_user_credentials`.
    2. **Variable names**: Some variable names, such as `conn`, `cursor`, and `result`, could be more descriptive.

    **Positive Observations**

    1. **Type hints**: The function uses type hints for the input parameters, which is a good practice.
    2. **Database connection**: The database connection is established using a try-except block, which is a good practice to handle potential errors.

    Here's an updated version of the code that addresses the critical issues and improves code quality:
    ```python
    import sqlite3
    from datetime import datetime

    def validate_user_credentials(username: str, password: str, db_path: str = 'users.db') -> tuple:
        """Validate user credentials against the database"""

        try:
            conn = sqlite3.connect(db_path)
            cursor = conn.cursor()

            query = "SELECT * FROM users WHERE username = ? AND password = ?"
            cursor.execute(query, (username, password))
            result = cursor.fetchone()

            if result:
                log_query = "INSERT INTO login_logs (username, timestamp) VALUES (?, ?)"
                cursor.execute(log_query, (username, datetime.now()))
                conn.commit()

                permissions = []
                perm_query = "SELECT permission FROM user_permissions WHERE user_id = ?"
                cursor.execute(perm_query, (result[0],))
                permissions.extend(cursor.fetchall())

                return True, permissions
            else:
                return False, None

        except sqlite3.Error as e:
            # Handle database errors
            print(f"Database error: {e}")
            return False, None

        finally:
            if conn:
                conn.close()
    ```
    Note that I've also added a `try-except` block to handle potential database errors and closed the connection in the `finally` block to ensure it's always closed.

Pretty slick, right? Make sure to exit the model.

.. code-block:: console

    /bye


**CHALLENGE** - Now create your own model file, load it into the model_data volume,
and create and run your custom model.

Recap
-----
In this lab, you:

- Customized a couple models based on **llama3.2:3b** and **codellama** and ran simple tests against them

This completes Module 1 working directly with models. Click Next to move on to Module 2.
