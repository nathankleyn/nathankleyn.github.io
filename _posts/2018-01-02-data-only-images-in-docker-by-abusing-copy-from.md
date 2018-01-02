---
layout: post
title: Data-Only Images In Docker By Abusing COPY --from
---

I recently employed a little trick to make Docker data-only images which can be joined into other Docker images as needed.

Image you have a set of configuration that you want to store somewhere separately to the code. Well, you can make a Docker image of _just_ this configuration very easily:

```dockerfile
FROM scratch
ADD ./ /fake-config
```

The only thing in this image will be the directory you added, making it as barebones as possible [^1].

Now, you may wonder what the use of an image like this is — after all, it can't be executed or run because it has no binaries inside. Enter the `COPY --from` command:

```dockerfile
FROM <some base image>
# ...
COPY --from=test/fake-config /fake-config /fake-config
# ...
```

With one command, the contents of your data-only image has been added to another image. This is super useful for adding configuration that you want shared between applications to the Docker images for them [^2].

[^1]: There is actually [some low-level plumbing within an image based on `scratch`](https://embano1.github.io/post/scratch/).
[^2]: This is especially true since you can't use symlinks within a Docker context, making it difficult to otherwise do this without some script to wrap the call to `docker build` that copies the common stuff around — messy.
