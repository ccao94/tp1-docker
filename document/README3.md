# TP 3

Q 3-1: Ansible Inventory and Base Commands

Inventory (ansible/inventories/setup.yml)
This file defines the remote servers (hosts) we want to manage.

all:
  vars:
    # The user we connect with
    ansible_user: admin
    # The path to our private key (relative to this file)
    ansible_ssh_private_key_file: ../keys/id_rsa 
  children:
    # We create a 'prod' group
    prod:
      hosts:
        # Our server's domain
        cao.cao.takima.cloud:

The ansible/keys/ directory is added to .gitignore to keep the private key secure.

___________________________________________________________________________________________________

Q 3-2: Playbook and Roles Documentation

to keep the project organized, the Docker installation logic was moved into an Ansible Role.

Role: ansible/roles/docker/
A role named docker was created using ansible-galaxy init. Its only job is to install and start Docker. The installation tasks were moved from the main playbook into ansible/roles/docker/tasks/main.yml.

Main Playbook (ansible/playbook.yml)
The main playbook is now much simpler. It just defines the target hosts (all) and calls the roles it needs to run.

- hosts: all
  gather_facts: true # The Docker role needs facts (e.g., OS release)
  become: true       # The Docker role needs sudo
roles: - docker

___________________________________________________________________________________________________

Q 3-3: docker_container Tasks Documentation

we use the docker_container module in separate roles to launch each service. The ansible_python_interpreter is set to /opt/docker_venv/bin/python because this module requires the Python docker SDK, which we installed in that virtual environment.

Role: database/tasks/main.yml

- name: Run database container
  docker_container:
    name: db
    image: ccao94/tp1-database:latest # Image from Docker Hub
    restart_policy: always
    networks:
      - name: app-network
    volumes:
      - "db-data:/var/lib/postgresql/data" # Use a persistent named volume
    env:
      # Inject environment variables for Postgres
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
    state: started
Role: backend/tasks/main.yml

- name: Run backend container
  docker_container:
    name: api
    image: ccao94/tp1-backend:latest
    restart_policy: always
    networks:
      - name: app-network
    env:
      # Inject environment variables for Spring Boot
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/db
      SPRING_DATASOURCE_USERNAME: usr
      SPRING_DATASOURCE_PASSWORD: pwd
    state: started
This role runs after the database role, ensuring the db hostname exists.

Role: httpd/tasks/main.yml

- name: Run httpd proxy container
  docker_container:
    name: web
    image: ccao94/tp1-httpd:latest
    restart_policy: always
    networks:
      - name: app-network
    ports:
      - "80:80" # The only port exposed to the host
    state: started

___________________________________________________________________________________________________

