# Using the Official MongoDB Image in the Memory Limits Lab

**The fix is already published in the course repository.** [See the commit](https://github.com/lm-academy/docker-course/commit/8a0cdf6a263fdff6d732c8ab8c792fcb926e4e84) for the exact changes, and pull the latest version if you downloaded the course files earlier.

**Lab: Setting Memory Limits for Containers** uses a MongoDB container to show how memory limits work. The recording runs the image `mongodb/mongodb-community-server:7.0-ubuntu2204`, which now refuses to start on machines running Linux kernel 6.19 or newer. Please use the Docker Official Image `mongo:7.0` instead. It runs the same MongoDB 7.0 series.

## What you would see with the old image

The container exits right after starting, and `docker logs mongodb` prints a message starting with:

```
ERROR: Detected Linux kernel
```

The image checks your machine's kernel before MongoDB even starts. MongoDB 7.0 itself works fine on these kernels; only this image's startup check blocks it. You can see your kernel version with `docker run --rm alpine uname -r`. Even if yours is older, please switch anyway so your setup matches the course files.

## What to change

Keep every command exactly as in the lecture, and replace only the image: wherever the lecture uses `mongodb/mongodb-community-server:7.0-ubuntu2204`, use `mongo:7.0`. For example:

```bash
docker run -d --rm --name mongodb --memory=20m mongo:7.0
```

If you downloaded the course files earlier and come across `mongo:7.0-ubuntu2204`, use `mongo:7.0` there too. That tag does not exist.

## What looks slightly different

- Memory usage is a little lower than in the recording: expect around 76 MiB instead of roughly 85 MB.
- With `--memory=20m`, the container is still killed for running out of memory, and `docker inspect mongodb` still shows `"OOMKilled": true`. The exit code can differ from the recording, and you will most likely see `137`.

Everything else in the lecture works exactly as shown.
