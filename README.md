
## EIDSR DOCKER SETUP


## Installation

This project stack and setup, requires:-
    
- [Docker](https://www.docker.com/)
- [Docker compose Engine](https://docs.docker.com/compose/)
    
## Project Setup

eIDSR App is made up of 2 services, namely;-

- DHIS2 as the data warehouse, manager, and backend.
-  Mediator, used for integration with national and referral hospitals and acting as the bridge between the national and referral hospitals and the DHIS2 API and data persistent layer.

Installation uses compose to manage all those 3 services and container   

 Docker compose setup, create a file named docker-compose.yml and add the following:-

```JS
version: '3.4'

services:

  mediator:
    image: udsmdhis2/eidsr-integration:latest
    restart: always
    env_file:
      - .env
    command: node main.js
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${SERVER_PORT}"]
      interval: 30s
      timeout: 30s

  db:
    image: postgis/postgis:12-3.3-alpine
    command: postgres -c max_locks_per_transaction=1000 -c max_connections=3000
    env_file:
      - .env
    restart: always
    volumes:
      - db-volume:/var/lib/postgresql/data

  dhis:
    image: dhis2/core:2.36.6
    restart: always
    ports:
      - 8080:8080
    env_file:
      - .env
    volumes:
      - ./conf:/DHIS2_home
    depends_on:
      - db
volumes:
  db-volume:
    driver: local
      
networks:
  default:
    external:
      name: eidsr-network
```
## Environment Variables

To run this project, you will need to add the following environment variables to your .env file

| KEY                     | DESCRIPTION                                                       |
| ----------------------- | ----------------------------------------------------------------- |
| DHIS2_HFR_URL           | eIDSR data warehouse and manager URL                              |
| DHIS2_HFR_USERNAME      | eIDSR EMR user to send data to eIDSR data warehouse               |
| DHIS2_HFR_PASSWORD      | eIDSR EMR password                                                |
| MIDDLEWARE_HFR_USERNAME | Mediator username required to authenticate EMR users sending data |
| MIDDLEWARE_HFR_PASSWORD | Rdsj424hb@3899!                                                   |
| EMAIL_HOST              | Mediator email SMTP host in case email sending is required        |
| EMAIL_USER              | Mediator email address for sending emails                         |
| EMAIL_PASSWORD          | Mediator email address''s password                                |
| EMAIL_RECIPIENTS        | Email recipients for system monitoring                            |
| SERVER_PORT             | Mediator internal port                                            |
| POSTGRES_USER           | DHIS2 postgres user                                               |
| POSTGRES_DB             | DHIS2 postgres database                                           |
| POSTGRES_PASSWORD       | DHIS2 postgres password                                           |
| PGTZ                    | DHIS2 postgres timezone data                                      |
| TZ                      | Mediator server timezone data                                     |
| DHIS_HOME               | DHIS2 home directory                                              |
| WAIT_FOR_DB_CONTAINER   | docker compose wait environment                                   |
| POSTGRES_HOST           | Postgres service name                                             | 

## DHIS2 Config

DHIS instance uses the dhis.conf file to define runtime environments. This file will reside in the conf directory with the following contents. 

```JS
# Hibernate SQL dialect
connection.dialect = org.hibernate.dialect.PostgreSQLDialect

# JDBC driver class
connection.driver_class = org.postgresql.Driver

# Database connection URL
connection.url = jdbc:postgresql://${POSTGRES_HOST}/${POSTGRES_DB}

# Database username
connection.username = ${POSTGRES_USERNAME}

# Database password (sensitive)
connection.password = ${POSTGRES_PASSWORD}

# Database schema behavior, can be 'validate', 'update', 'create', 'create-drop
connection.schema = update

# Max size of connection pool (default: 40)
connection.pool.max_size = 1024

# Encryption password (sensitive)
encryption.password = xxxx

```
## Running

Step 1.
- We create a network for the eidsr docker orchastration

```BASH
docker network create eidsr-network
```

Step 2.
- We bring up all the services by Running

```BASH
docker-compose up -d
```

`NOTE`: It is advisable to monitor logs as the services start. This helps to catch any issues earlier in service start up.
## Logs

You can view logs of a particular container/services with either docker or docker-compose. Viewing logs with docker-compose, requires the service name and for you to be in the root directory of the project setup.

- Logs with docker

```BASH

docker logs -f --tail <NUMBER OF PREVIOUS LOGS> <DOCKER CONTAINER NAME OR ID>

```

- Logs with docker-compose

```BASH

docker-compose logs -f --tail <NUMBER OF PREVIOUS LOGS> <DOCKER SERVICE NAME>

```

With `docker-compose`, you can also view logs of all containers in a particular stack by running the `docker-compose` logs command without the service name


## License

[MIT](https://choosealicense.com/licenses/mit/)

```JS
MIT License

Copyright (c) 2022 UDSM DHIS2

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
