# Open Reaction Database: App (ord-app)

This repository contains a fastapi+postrgresql API and web-app for editing records in [Open Reaction Database](https://docs.open-reaction-database.org).

## Installation

### IMPORTANT: Install ord-data and ord-schema first!!

Install PostgreSQL with the RDKIT plugin and populate a database by installing `git-lfs` before cloning [`ord-data`](https://github.com/open-reaction-database/ord-data), which will download all the protobuf files from git-LFS.

```shell
$ sudo apt install -y postgresql-common
$ sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
$ sudo apt install --update --upgrade postgresql-17 postgresql-17-rdkit postgresql-17-pgvector
```

1. install and initialize git-lfs
2. clone `ord-data` 
3. clone `ord-schema`
4. create and activate `ord-schema/.venv/`
5. install `ord-schema` in .venv
6. run expore_data.py

```bash
$ curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.python.sh | bash
$ git lfs install
$ git clone https://github.com/open-reaction-database/ord-data
$ git clone https://github.com/hobson/ord-schema
$ cd ord-schema
$ uv venv -p 3.12
$ source .venv/bin/activate
$ uv pip install -e .[docs,tests,examples]
$ python explore_data.py
$ cd ..
```

I also needed to update my search path to include the `ord` schema where I created and loaded all my tables.

```shell
psql -d ord -u hobs 'SET search_path = ag_catalog, "$user", public, ord, pubchem;'
```

### Install `ord-app`

```bash
$ sudo apt install -y postgresql-common
$ sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
$ sudo apt install --update --upgrade postgresql-17 postgresql-rdkit
$ git clone git@github.com:hobson/ord-app.git
$ cd ord-app
$ uv pip install -e ".[tests]"
```


```shell
cd src/ord_app/service_api
ORD_APP_TESTING=TRUE fastapi dev main.py
```
    
This creates a test PostgreSQL database and starts the server at http://localhost:8000. Navigate to
http://localhost:8000/docs for the interactive Swagger docs.

## Run in Docker

### docker-compose
You can run the Back-End and the Database using a single docker-compose file.
```shell
docker compose up -d
```
At the same time, you need to run the Front-End separately.
```shell
cd ui
```

```shell
npm ci
```

```shell
npm run dev
```

### Single docker file
Or run the Front-End and Back-End in a single Dockerfile.

_Note: the database must be on the same network as docker or docker must connect to the external database (and have access)_

1. Build the Docker image
   ```shell
   docker build -f Dockerfile.single -t ord . 
   ```
2. Run the Docker image
   ```shell
   docker run \
   --network ord_network \
   -e VITE_API_ENDPOINT="http://localhost:8000/service_api/api/v1" \
   -e VITE_AUTH0_DOMAIN="..." \
   -e VITE_AUTH0_CLIENT_ID="..." \
   -e VITE_AUTH0_AUDIENCE="..." \
   -e VITE_AUTH0_ISSUER="..." \
   -e PG_DSN="postgresql+asyncpg://ord@db:5432/ord"
   --rm -p 5173:5173 -p 8000:8000 ord
   ```

Envs for backend:

| Name                    | Description                                        | Required | Default                                                 |
|-------------------------|----------------------------------------------------|----------|---------------------------------------------------------|
| `pg_dsn`                | DSN for connecting to the database                 | false    | `postgresql+psycopg://ord@localhost:5400/ord`           |
| `cors_origins`          | Allowed origins                                    | false    | `["http://localhost:5173"]`                             |
| `app_env`               | Manages the application context (debug parameters) | false    | `production` (available: `localhost`, `production`)     |
| `vite_auth0_domain`     | Auth0 config                                       | true     | -                                                       |
| `vite_auth0_algorithms` | Auth0 config                                       | true     | -                                                       |
| `vite_auth0_audience`   | Auth0 config                                       | true     | -                                                       |
| `vite_auth0_issuer`     | Auth0 config                                       | true     | -                                                       |
| `vite_auth0_client_id`  | Auth0 config                                       | true     | -                                                       |


## Testing

Python tests are written with `pytest`:

```shell
pytest -vv
```
