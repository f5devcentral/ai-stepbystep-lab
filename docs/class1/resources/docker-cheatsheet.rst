Docker Cheat Sheet
==================

Container Lifecycle Management
------------------------------

Creating Containers
^^^^^^^^^^^^^^^^^^^

**From an image:**

.. code-block:: bash

   # Create container without starting
   docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

   # Create and start container
   docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

   # Run interactively with TTY
   docker run -it ubuntu:latest /bin/bash

   # Run in detached mode (background)
   docker run -d nginx:latest

   # Run with custom name
   docker run --name my-container nginx:latest

Starting Containers
^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Start stopped container
   docker start CONTAINER_ID_OR_NAME

   # Start multiple containers
   docker start container1 container2

   # Start and attach to container
   docker start -a CONTAINER_ID_OR_NAME

Restarting Containers
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Restart container
   docker restart CONTAINER_ID_OR_NAME

   # Restart with timeout (default 10s)
   docker restart -t 30 CONTAINER_ID_OR_NAME

Stopping Containers
^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Stop container gracefully
   docker stop CONTAINER_ID_OR_NAME

   # Stop with timeout (default 10s)
   docker stop -t 30 CONTAINER_ID_OR_NAME

   # Force stop (SIGKILL)
   docker kill CONTAINER_ID_OR_NAME

   # Stop all running containers
   docker stop $(docker ps -q)

Destroying Containers
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Remove container (must be stopped first)
   docker rm CONTAINER_ID_OR_NAME

   # Force remove running container
   docker rm -f CONTAINER_ID_OR_NAME

   # Remove multiple containers
   docker rm container1 container2

   # Remove all stopped containers
   docker container prune

   # Remove container automatically when it exits
   docker run --rm IMAGE

Port Mapping
------------

Basic Port Mapping
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Map container port to host port
   docker run -p HOST_PORT:CONTAINER_PORT IMAGE

   # Example: Map container port 80 to host port 8080
   docker run -p 8080:80 nginx:latest

   # Map to specific host interface
   docker run -p 127.0.0.1:8080:80 nginx:latest

   # Map to all interfaces (default)
   docker run -p 0.0.0.0:8080:80 nginx:latest

Advanced Port Options
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Map random host port
   docker run -P IMAGE

   # Map multiple ports
   docker run -p 8080:80 -p 8443:443 nginx:latest

   # Map UDP port
   docker run -p 5353:53/udp IMAGE

   # Map port range
   docker run -p 8000-8010:8000-8010 IMAGE

Volume Management
-----------------

Bind Mounts
^^^^^^^^^^^

.. code-block:: bash

   # Mount host directory to container
   docker run -v /host/path:/container/path IMAGE

   # Mount as read-only
   docker run -v /host/path:/container/path:ro IMAGE

   # Mount current directory
   docker run -v $(pwd):/app IMAGE

Named Volumes
^^^^^^^^^^^^^

.. code-block:: bash

   # Create named volume
   docker volume create my-volume

   # Use named volume
   docker run -v my-volume:/data IMAGE

   # List volumes
   docker volume ls

   # Inspect volume
   docker volume inspect my-volume

   # Remove volume
   docker volume rm my-volume

   # Remove unused volumes
   docker volume prune

Temporary Filesystems
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Create tmpfs mount (in-memory)
   docker run --tmpfs /tmp IMAGE

   # Tmpfs with size limit
   docker run --tmpfs /tmp:size=100m IMAGE

Container Information
---------------------

Listing Containers
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # List running containers
   docker ps

   # List all containers (including stopped)
   docker ps -a

   # List container IDs only
   docker ps -q

   # List with custom format
   docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

Inspecting Containers
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Get detailed container information
   docker inspect CONTAINER_ID_OR_NAME

   # Get specific information using format
   docker inspect --format='{{.NetworkSettings.IPAddress}}' CONTAINER

   # View container logs
   docker logs CONTAINER_ID_OR_NAME

   # Follow log output
   docker logs -f CONTAINER_ID_OR_NAME

   # Show timestamps in logs
   docker logs -t CONTAINER_ID_OR_NAME

Executing Commands
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Execute command in running container
   docker exec CONTAINER_ID_OR_NAME COMMAND

   # Interactive shell
   docker exec -it CONTAINER_ID_OR_NAME /bin/bash

   # Execute as specific user
   docker exec -u root CONTAINER_ID_OR_NAME COMMAND

Image Management
----------------

Working with Images
^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Pull image from registry
   docker pull IMAGE:TAG

   # List local images
   docker images

   # Remove image
   docker rmi IMAGE:TAG

   # Remove unused images
   docker image prune

   # Build image from Dockerfile
   docker build -t IMAGE_NAME:TAG PATH

Docker Compose
--------------

Basic Commands
^^^^^^^^^^^^^^

.. code-block:: bash

   # Start services defined in docker-compose.yml
   docker-compose up

   # Start in detached mode
   docker-compose up -d

   # Start specific service
   docker-compose up SERVICE_NAME

   # Stop services
   docker-compose down

   # Stop and remove volumes
   docker-compose down -v

Service Management
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Start stopped services
   docker-compose start

   # Stop running services
   docker-compose stop

   # Restart services
   docker-compose restart

   # View service logs
   docker-compose logs

   # Follow logs for specific service
   docker-compose logs -f SERVICE_NAME

Scaling and Building
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Scale service to multiple instances
   docker-compose up --scale SERVICE_NAME=3

   # Build services
   docker-compose build

   # Build specific service
   docker-compose build SERVICE_NAME

   # Pull latest images
   docker-compose pull

Sample docker-compose.yml
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

   version: '3.8'

   services:
     web:
       image: nginx:latest
       ports:
         - "8080:80"
       volumes:
         - ./html:/usr/share/nginx/html:ro
         - web-logs:/var/log/nginx
       environment:
         - NGINX_HOST=localhost
       depends_on:
         - db

     db:
       image: postgres:13
       environment:
         POSTGRES_DB: myapp
         POSTGRES_USER: user
         POSTGRES_PASSWORD: password
       volumes:
         - db-data:/var/lib/postgresql/data
       ports:
         - "5432:5432"

     app:
       build: .
       ports:
         - "3000:3000"
       volumes:
         - .:/app
         - /app/node_modules
       environment:
         - NODE_ENV=development
       depends_on:
         - db

   volumes:
     db-data:
     web-logs:

Networking
----------

Network Commands
^^^^^^^^^^^^^^^^

.. code-block:: bash

   # List networks
   docker network ls

   # Create network
   docker network create my-network

   # Connect container to network
   docker network connect my-network CONTAINER

   # Disconnect container from network
   docker network disconnect my-network CONTAINER

   # Remove network
   docker network rm my-network

Network Types
^^^^^^^^^^^^^

.. code-block:: bash

   # Bridge network (default)
   docker run --network bridge IMAGE

   # Host network (use host's network stack)
   docker run --network host IMAGE

   # No network
   docker run --network none IMAGE

   # Custom network
   docker run --network my-network IMAGE

Cleanup Commands
----------------

System Cleanup
^^^^^^^^^^^^^^

.. code-block:: bash

   # Remove unused containers, networks, images, and cache
   docker system prune

   # Remove everything including volumes
   docker system prune -a --volumes

   # Show disk usage
   docker system df

Specific Cleanup
^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Remove stopped containers
   docker container prune

   # Remove unused images
   docker image prune

   # Remove unused volumes
   docker volume prune

   # Remove unused networks
   docker network prune

Common Use Cases
----------------

Development Environment
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Run with volume mount for live code editing
   docker run -it -v $(pwd):/app -w /app node:16 bash

   # Web server with port mapping and volume
   docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx

Database with Persistent Storage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # PostgreSQL with named volume
   docker run -d \
     --name postgres-db \
     -e POSTGRES_PASSWORD=mypassword \
     -v postgres-data:/var/lib/postgresql/data \
     -p 5432:5432 \
     postgres:13

Temporary Testing Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Auto-remove container when done
   docker run --rm -it ubuntu:latest bash

Environment Variables
---------------------

Setting Environment Variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Single environment variable
   docker run -e VAR_NAME=value IMAGE

   # Multiple variables
   docker run -e VAR1=value1 -e VAR2=value2 IMAGE

   # Load from file
   docker run --env-file .env IMAGE

Useful Options
--------------

Common Run Options
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Complete example with common options
   docker run -d \
     --name my-app \
     --restart unless-stopped \
     -p 8080:80 \
     -v /host/data:/app/data \
     -e ENV=production \
     --memory=512m \
     --cpus=1.0 \
     nginx:latest

Resource Limits
^^^^^^^^^^^^^^^

.. code-block:: bash

   # Memory limit
   docker run --memory=512m IMAGE

   # CPU limit
   docker run --cpus=1.5 IMAGE

   # CPU and memory together
   docker run --memory=1g --cpus=2.0 IMAGE

Restart Policies
^^^^^^^^^^^^^^^^

.. code-block:: bash

   # Never restart
   docker run --restart=no IMAGE

   # Always restart
   docker run --restart=always IMAGE

   # Restart unless stopped
   docker run --restart=unless-stopped IMAGE

   # Restart on failure (up to 3 times)
   docker run --restart=on-failure:3 IMAGE