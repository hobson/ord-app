# ord-app

## Local development

1. Install dependencies

   * PostgreSQL
   * Python >= 3.10
   * Git LFS

2. Clone `ord-schema` and `ord-data` and use them to create and populate a postgresql database

I also needed to update my search path to include the `ord` schema where I created and loaded all my tables.

```shell
psql -d ord -u hobs 'SET search_path = ag_catalog, "$user", public, ord, pubchem;'
```

3. Install the package

```shell
git clone git@github.com:open-reaction-database/ord-app.git
cd ord-app
pythom -m pip install -e ".[tests]"
```

4. Run the FastAPI server

```shell
# cd src/ord_app/service_api
ORD_APP_TESTING=TRUE fastapi dev src/ord_app/service_api/main.py
```

This creates a test PostgreSQL database and starts the server at http://localhost:8000. Navigate to
http://localhost:8000/docs for the interactive Swagger docs.

### RelationNotFound
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
