Q 1-1: Using ENV hard-codes the secret into the image layer, making it visible to anyone with the image. The -e flag injects the secret only at runtime, keeping the image secure and reusable.
_____________________________________________________________________________________________________

Q 1-2: Containers are ephemeral (temporary); their files are deleted when the container is removed. A volume stores the database data on the host machine, ensuring the data persists even if the container is destroyed.
_____________________________________________________________________________________________________

Q 1-3: Database container essentials
(database/Dockerfile)

FROM postgres:17.2-alpine

COPY ./init/ /docker-entrypoint-initdb.d/

Commands All build and run commands are managed by the docker-compose.yml file (see Q 1-8).

_____________________________________________________________________________________________________

Q 1-4: we use multistage builds to keep the final image small and secure.

- 1 (build): Uses a heavy image (with JDK & Maven) to compile the .jar file.

- 2 (run): Uses a lightweight image (with JRE only).

- 3 (copy) : We copy only the compiled .jar from 1 into 2, discarding all build tools.

_____________________________________________________________________________________________________

Q 1-5: it acts as a single, secure "front door" for our application. It hides the backend services from the user and can manage tasks like SSL termination, load balancing, and serving static files.

_____________________________________________________________________________________________________

Q 1-6: it defines and runs multi-container applications from a single YAML file. It manages the entire application lifecycle (start, stop), dependencies, and networking with one command.

____________________________________________________________________________________________________

Q 1-7: Docker-compose most important commands

- docker compose up -d: Starts all services in the background.

- docker compose down: Stops and removes all services and networks.

- docker compose logs -f: Tails (follows) the logs of all services.

- docker compose ps: Lists the running services.

- docker compose build: Forces a rebuild of the images.

_____________________________________________________________________________________________________

Q 1-8: docker-compose.yml documentation
The docker-compose.yml file defines our 3-tier application:

version: '3.8'

services:
  # Backend API
  backend:
    build: ./backend
    container_name: api
    restart: unless-stopped
    networks:
      - app-network
    depends_on:
      - database
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://database:5432/db
      - SPRING_DATASOURCE_USERNAME=usr
      - SPRING_DATASOURCE_PASSWORD=pwd
      
  # Database
  database:
    build: ./database
    container_name: db
    restart: unless-stopped
    networks:
      - app-network
    environment:
      - POSTGRES_DB=db
      - POSTGRES_USER=usr
      - POSTGRES_PASSWORD=pwd
    volumes:
      - db-data:/var/lib/postgresql/data
      
  # Reverse Proxy
  httpd:
    build: ./httpd
    container_name: web
    restart: unless-stopped
    ports:
      - "80:80"
    networks:
      - app-network
    depends_on:
      - backend

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
    driver: local

_____________________________________________________________________________________________________

Q 1-9: Document publication commands

1. Login docker login

2. Tag Images

docker tag tp1/database ccao94/tp1-database:1.0
docker tag tp1/backend ccao94/tp1-backend:1.0
docker tag tp1/httpd ccao94/tp1-httpd:1.0

3. Push Images

docker push ccao94/tp1-database:1.0
docker push ccao94/tp1-backend:1.0
docker push ccao94/tp1-httpd:1.0
_____________________________________________________________________________________________________

Q 1-10: we use a remote repository (like Docker Hub) to share images with a team, version our builds, and deploy them consistently across different environments (e.g., development, testing, production).