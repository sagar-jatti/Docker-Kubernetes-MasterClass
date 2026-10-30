# Updating the MongoDB Image and Credentials for the Notes App

**The fix is already published in the course repository.** [See the commit](https://github.com/lm-academy/docker-course/commit/8a0cdf6a263fdff6d732c8ab8c792fcb926e4e84) for the exact changes, and pull the latest version if you downloaded the course files earlier.

Both backends in this project run their own MongoDB database. The recordings use the image `mongodb/mongodb-community-server:7.0-ubuntu2204`, which refuses to start on machines running Linux kernel 6.19 or newer. We'll use the Docker Official Image `mongo:7.0` instead. It runs the same MongoDB 7.0 series, and it expects different names for the root credentials:

- **Image:** `mongodb/mongodb-community-server:7.0-ubuntu2204` becomes `mongo:7.0`
- **Root username variable:** `MONGODB_INITDB_ROOT_USERNAME` becomes `MONGO_INITDB_ROOT_USERNAME`
- **Root password variable:** `MONGODB_INITDB_ROOT_PASSWORD` becomes `MONGO_INITDB_ROOT_PASSWORD`

**Why the variable names matter:** the official image silently ignores the old names. The database starts and your backend connects, but no root user is created and authentication stays off.

## What to change

**Configuring Docker Compose for Notebooks Services** and **Setting Up Docker Compose for Notes Services:** set the image of both database services:

```yaml
  notebooks-db:
    image: mongo:7.0
```

```yaml
  notes-db:
    image: mongo:7.0
```

In each backend's `.env` file, use the `MONGO_` names:

```
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=rootpassword
```

Everything else stays exactly as shown, including the init scripts, the database users, and `DB_URL`. In the course repository, the files under `notes-rest-api/notebooks-backend/` and `notes-rest-api/notes-backend/` already use these values.

## Check that it worked

After `docker compose up -d`:

- `docker compose logs notebooks-db` shows the `INITIALIZING` message from your init script.
- `docker compose exec notebooks-db mongosh`, followed by `show dbs`, prints `MongoServerError[Unauthorized]: Command listDatabases requires authentication`.

If `show dbs` lists databases instead, authentication is off. Fix the variable names, then run `docker compose down -v` and `docker compose up -d` so MongoDB starts from a fresh data directory. The same checks apply to `notes-db`.
