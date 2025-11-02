# TP 3

Q 3-1: Ansible Inventory and Base Commands

Inventory (ansible/inventories/setup.yml)
This file defines the remote servers (hosts) we want to manage.

all:
  vars:
    # the user we connect with
    ansible_user: admin
    # the path to our private key
    ansible_ssh_private_key_file: ansible/keys/id_rsa 
  children:
    # we create a 'prod' group
    prod:
      hosts:
        # our server's domain
        cao.cao.takima.cloud:

The ansible/keys/ directory is added to .gitignore to keep the private key secure.

___________________________________________________________________________________________________

Q 3-2: Playbook and Roles Documentation

to keep the project organized, logic is split into Roles. The main playbook (playbook.yml) just calls these roles in the correct order.

- name: Install Docker SDK for Python (Globally and break protection)
  command: pip3 install docker requests docker-compose --break-system-packages

Main Playbook (ansible/playbook.yml) : this file loads our encrypted vault.yml file (which holds all passwords) and then runs the 5 roles in sequence.

- hosts: all
  gather_facts: true
  become: true

  vars_files:
    - vault.yml # load our encrypted passwords

  roles:
    - docker # install Docker & Python libs
    - network # create Docker networks
    - database # launch PostgreSQL container
    - backend # launch backend container
    - httpd # launch Apache reverse proxy

___________________________________________________________________________________________________

Q 3-3: docker_container Tasks Documentation

we use the docker_container module to launch our 3 services. All secrets (passwords, db names) are loaded securely from vault.yml using variables like {{ postgres_password }}.
we use 2 separate Docker networks for security:

- app-network: for internal communication between the backend and database.

- proxy-network: for communication between the httpd proxy and the backend.


Role: database/tasks/main.yml (this container only connects to the internal app-network)

- name: Launch database container
  docker_container:
    name: db
    image: ccao94/tp1-database:latest
    networks:
      - name: app-network
    volumes:
      - "db-data:/var/lib/postgresql/data"
    env:
      # Use variables from Ansible Vault
      POSTGRES_USER: "{{ postgres_user }}"
      POSTGRES_PASSWORD: "{{ postgres_password }}"
      POSTGRES_DB: "{{ postgres_db }}"
    state: started
    restart_policy: on-failure
  vars:
    ansible_python_interpreter: /usr/bin/python3


Role: backend/tasks/main.yml (this container connects to both networks : app-network to talk to the DB, and proxy-network to be reached by the proxy)

- name: Launch backend container
  docker_container:
    name: backend #named 'backend' to match httpd.conf
    image: ccao94/tp1-backend:latest
    restart_policy: on-failure
    networks:
      - name: app-network
      - name: proxy-network 
    env:
      # Use variables from Ansible Vault
      SPRING_DATASOURCE_URL: "jdbc:postgresql://db:5432/{{ postgres_db }}"
      SPRING_DATASOURCE_USERNAME: "{{ postgres_user }}"
      SPRING_DATASOURCE_PASSWORD: "{{ postgres_password }}"
  vars:
    ansible_python_interpreter: /usr/bin/python3


Role: httpd/tasks/main.yml (This role includes cleanup tasks and only connects to the proxy-network)

- name: Force stop and remove ANY old web containers
  docker_container:
    name: "{{ item }}"
    state: absent
  loop: [ 'web', 'proxy' ]
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Launch Apache reverse proxy
  docker_container:
    name: web
    image: ccao94/tp1-httpd:latest
    networks:
      - name: proxy-network
    ports:
      - "80:80" # the only port exposed to the internet
    state: started
    restart_policy: on-failure
  vars:
    ansible_python_interpreter: /usr/bin/python3

___________________________________________________________________________________________________


Q " Is it really safe to deploy automatically every new image on the hub ? explain. What can I do to make it more secure?"

no, it's not safe. Automated tests can miss critical bugs and break production.

To make it safer, stop using :latest and deploy specific versions (like 1.1.0). Always deploy to a "staging" server for manual checks first, then add a manual approval step in github actions before the final production run.
