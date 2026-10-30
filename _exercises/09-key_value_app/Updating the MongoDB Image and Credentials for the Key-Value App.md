# Updating the MongoDB Image and Credentials for the Key-Value App

**The fix is already published in the course repository.** [See the commit](https://github.com/lm-academy/docker-course/commit/8a0cdf6a263fdff6d732c8ab8c792fcb926e4e84) for the exact changes, and pull the latest version if you downloaded the course files earlier.

This project runs MongoDB in a container. The recordings use the image `mongodb/mongodb-community-server:7.0-ubuntu2204`, which now refuses to start on machines running Linux kernel 6.19 or newer. We'll use the Docker Official Image `mongo:7.0` instead. It runs the same MongoDB 7.0 series.

The official image expects different names for the root credentials, so there are two changes, and you need both:

- **Image:** `mongodb/mongodb-community-server:7.0-ubuntu2204` becomes `mongo:7.0`
- **Root username variable:** `MONGODB_INITDB_ROOT_USERNAME` becomes `MONGO_INITDB_ROOT_USERNAME`
- **Root password variable:** `MONGODB_INITDB_ROOT_PASSWORD` becomes `MONGO_INITDB_ROOT_PASSWORD`

**Why the variable names matter:** the official image silently ignores the old names. MongoDB still starts and your backend still connects, but no root user is created and authentication stays off. Anyone who can reach port 27017 can read your data, and nothing warns you.

## Lecture by lecture

**Running a MongoDB Server.** When we search Docker Hub, pick `mongo` (the Docker Official Image) instead of MongoDB Community Server, and use the tag `7.0`:

```bash
docker run -d --name mongodb mongo:7.0
docker exec -it mongodb mongosh
```

**Adding Root Credentials to MongoDB.** In `start-db.sh`, keep the variable names `MONGODB_IMAGE` and `MONGODB_TAG` and change only their values:

```bash
MONGODB_IMAGE="mongo"
MONGODB_TAG="7.0"
```

In the `docker run` command, use the `MONGO_` names:

```bash
  -e MONGO_INITDB_ROOT_USERNAME=$ROOT_USER \
  -e MONGO_INITDB_ROOT_PASSWORD=$ROOT_PASSWORD \
```

When we check authentication with `docker exec -it mongodb mongosh` and `show dbs`, you should get the same error as in the recording:

```
MongoServerError[Unauthorized]: Command listDatabases requires authentication
```

If you see a list of databases instead (`admin`, `config`, `local`), authentication is off. Double-check the variable names.

**Defining Ports, Networks, and Volumes.** When we start a second container to connect with `mongosh` over the network, use `mongo:7.0` as its image too. Keep the rest of the command as in the lecture:

```bash
docker run -it --rm --network key-value-net mongo:7.0 mongosh ...
```

Everything else in the project works exactly as shown, including the init script, the volume, the network, and the backend. In the course repository, `key-value-app/start-db.sh` already uses these values.

## If something looks off

- **The `mongodb` container disappears right after `./start-db.sh`.** You are still on the old image. The script starts the container with `--rm`, so Docker removes it the moment it exits, and `docker logs mongodb` can no longer show you why. Switch the image as shown above.
- **`show dbs` still lists databases after you fixed the names.** MongoDB creates the root user only the first time it starts with an empty data directory. Once we add the `key-value-data` volume, an earlier start with the old names leaves that volume without a root user. Run `./cleanup.sh` (or `docker volume rm key-value-data`) and start the database again.
