# TP 1

Q 1-1: Using ENV saves the secret (like a password) directly into the image. Anyone who gets the image can see it. Using the -e flag only adds the secret when the container runs. This keeps the image secure.
_____________________________________________________________________________________________________

Q 1-2: Containers are temporary. When you delete a container, all its data is lost. A volume saves the data on the host computer, outside the container. This way, the data is safe even if the container is deleted.
_____________________________________________________________________________________________________

Q 1-3: Database container essentials
(database/Dockerfile)

FROM postgres:17.2-alpine

COPY ./init/ /docker-entrypoint-initdb.d/

Commands All build and run commands are managed by the docker-compose.yml file.

_____________________________________________________________________________________________________

Q 1-4: multistage builds make our final image small and secure.

- Stage 1 (build): We use a big image (with JDK & Maven) to build the app.

- Stage 2 (run): We use a small image (with JRE only) to run the app.

- COPY: We copy only the final .jar file from Stage 1 into Stage 2 and throw away all the heavy build tools.

_____________________________________________________________________________________________________

Q 1-5: a reverse proxy is the "front door" for our app. Users only connect to the proxy, which hides our backend services. This is more secure. It can also manage other tasks (like load balancing) from one single place.

_____________________________________________________________________________________________________

Q 1-6: it's important because it lets us define and run a full application (with many containers) from one single file. It manages starting, stopping, and connecting all the services with one command (docker compose up).

____________________________________________________________________________________________________

Q 1-7: Docker-compose most important commands

- docker compose up -d: starts all services in the background.

- docker compose down: stops and removes all services and networks.

- docker compose logs -f: shows the logs from all services (and "follows" new logs).

- docker compose ps: lists the running services.

- docker compose build: forces Docker to rebuild the images.

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

Q 1-10: we put images in an online repo (like Docker Hub) to share them with teammates. It also lets us save different versions (like 1.0, 1.1) and use the exact same image for development, testing, and production.


