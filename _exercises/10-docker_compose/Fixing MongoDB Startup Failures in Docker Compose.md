# Fixing MongoDB Startup Failures in Docker Compose

**The fix is already published in the course repository.** [See the commit](https://github.com/lm-academy/docker-course/commit/8a0cdf6a263fdff6d732c8ab8c792fcb926e4e84) for the exact changes, and pull the latest version if you downloaded the course files earlier.

The next lectures run MongoDB with Docker Compose. The recordings use the image `mongodb/mongodb-community-server:7.0-ubuntu2204`, which refuses to start on machines running Linux kernel 6.19 or newer. You can check your kernel version with `docker run --rm alpine uname -r`. We'll use the Docker Official Image `mongo:7.0` instead. It runs the same MongoDB 7.0 series.

## How the problem shows up

- `docker compose ps` shows the database service as `Exited (1)`.
- `docker compose logs db` shows a single line starting with `ERROR: Detected Linux kernel`.
- Once the backend joins the stack, it stays `Up` but never connects. After about 30 seconds it logs `MongooseServerSelectionError: getaddrinfo ENOTFOUND db`. That looks like a networking problem, but the real cause is the stopped database.

## The fix: two changes, and you need both

- **Image:** `mongodb/mongodb-community-server:7.0-ubuntu2204` becomes `mongo:7.0`
- **Root username variable:** `MONGODB_INITDB_ROOT_USERNAME` becomes `MONGO_INITDB_ROOT_USERNAME`
- **Root password variable:** `MONGODB_INITDB_ROOT_PASSWORD` becomes `MONGO_INITDB_ROOT_PASSWORD`

**Why the variable names matter:** the official image silently ignores the old names. MongoDB starts and your backend connects, but no root user is created and authentication stays off.

**Lab: Running MongoDB with Docker Compose.** Set the image of the `db` service:

```yaml
services:
  db:
    image: mongo:7.0
```

When we start a separate container to connect with `mongosh`, use `mongo:7.0` there as well, and keep the rest of the command as in the lecture:

```bash
docker run -it --rm --network compose_default mongo:7.0 mongosh ...
```

**Lab: Using Environment Variables in Docker Compose.** Use the `MONGO_` names, both in the `environment` example and in `.env.db-root-creds`:

```bash
export MONGO_INITDB_ROOT_USERNAME=root
export MONGO_INITDB_ROOT_PASSWORD=root_password
```

Everything else in the section works exactly as shown. In the course repository, `compose/compose.yaml` and `compose/.env.db-root-creds` already use these values.

## Check that authentication is on

With the credentials in place, run `docker compose exec db mongosh` and then `show dbs`. You should see:

```
MongoServerError[Unauthorized]: Command listDatabases requires authentication
```

If you see a list of databases instead, authentication is off. MongoDB creates the root user only the first time it starts with an empty data directory, so after fixing the names, run `docker compose down -v` to remove the old data and `docker compose up -d` to start fresh.
