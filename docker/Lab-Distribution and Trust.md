# Lab: Distribution and Trust

> **Difficulty**: Intermediate

> **Time**: Approximately 30 minutes

This lab focuses on understanding and securing image distribution. We'll
start with a simple `docker pull` and build up to using Docker Content Trust (DCT).

You will complete the following steps as part of this lab.

- [Step 1 - Pulling images by tag](#tag)
- [Step 2 - Pulling images by digest](#digest)
- [Step 3 - Docker Content Trust](#trust)
- [Step 4 - Official Images](#official)

# Prerequisites

You will need all of the following to complete this lab:

- A Docker Host running Docker 1.10 or higher (preferably Docker 1.12 or higher).
- This lab uses environment variables which can be clunky when working with `sudo`. Therefore none of the Docker commands in this particular lab are preceded with `sudo`.

# <a name="tag"></a>Step 1: Pulling images by tag

The most common and basic way to pull Docker images is by `tag`. The is where you specify an image name followed by an alphanumeric tag. The image name and tag are separated by a colon `:`. For example:

   ```
   $ docker pull alpine:edge
   ```

This command will pull the Alpine image tagged as `edge`. The corresponding image can be found [here on Docker Store](https://store.docker.com/images/alpine).

If no tag is specified, Docker will pull the image with the `latest` tag.

1. Pull the Alpine image with the `edge` tag.

   ```
   $ docker pull alpine:edge

   edge: Pulling from library/alpine
   e587fa4f6e1f: Pull complete
   Digest: sha256:e5ab6f0941eb01c41595d35856f16215021a941e9893501d632ed4c0ee4e53a6
   Status: Downloaded newer image for alpine:edge
   ```

2. Confirm that the pull was successful.

   ```
   $ docker images

   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   alpine              edge                3fc33d6d5e74        4 weeks ago         4.799 MB
   ```

3. Run a new container from the image and ping www.docker.com.

   ```
   $ docker run --rm -it alpine:edge sh

   / # ping www.docker.com
   PING www.docker.com (104.239.220.248): 56 data bytes
   64 bytes from 104.239.220.248: seq=0 ttl=40 time=94.424 ms
   64 bytes from 104.239.220.248: seq=1 ttl=40 time=94.549 ms
   64 bytes from 104.239.220.248: seq=2 ttl=40 time=94.455 ms
   ^C
   ```

4. Exit the container.

In this step you pulled an image by tag and verified the pull by running a container from it.

# <a name="digest"></a>Step 2: Pulling images by digest

Pulling by tag is easy and convenient. However, tags are mutable, and the same tag can refer to different images over time. For example, you can add updates to an image and push the updated image using the same tag as a previous version of the image. This scenario where a single tag points to multiple versions of an image can lead to bugs and vulnerabilities in your production environments.

This is why pulling by digest is such a powerful operation. Thanks to the content-addressable storage model used by Docker images, we can target pulls to specific image contents by pulling by digest. In this step you'll see how to pull by digest.

1.  Pull the Alpine image with the `sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7` digest.

   ```
   $ docker pull alpine@sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7

   sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7: Pulling from library/alpine
   6589332ef57c: Pull complete
   Digest: sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7
   Status: Downloaded newer image for alpine@sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7
   ```

2. Check that the pull succeeded.

   ```
   $ docker images --digests alpine

   REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
   alpine              edge                <none>                                                                    3fc33d6d5e74        4 weeks ago         4.799 MB
   alpine              <none>              sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7   79c898d40088        7 weeks ago         4.799 MB
   ```

   Notice that there are now two Alpine images in the Docker Hosts local repository. One lists the `edge` tag. The other lists `<none>` as the tag along with the `b7233daf...` digest.

The content addressable storage model used by Docker images means that any changes made to an image will result in a new digest for the updated image. This means it is not possible for two images with different contents to have the same digest.

In this step you learned how to pull images by digest and list image digests using the `docker images` command. For more information about the `docker pull` command, see the [documentation](https://docs.docker.com/engine/reference/commandline/pull/).

# Step 3: Docker Content Trust

It's not easy to find the digest of a particular image tag. This is because it is computed from the hash of the image contents and stored in the image manifest. The image manifest is then stored in the Registry. This is why we needed a `docker pull` by tag to find digests previously. It would also be desirable to have additional security guarantees such as image freshness.

Enter Docker Content Trust: a system currently in the Docker Engine that verifies the publisher of images without sacrificing usability. Docker Content Trust implements [The Update Framework](https://theupdateframework.github.io/) (TUF), an NSF-funded research project succeeding Thandy of the Tor project. TUF uses a key hierarchy to ensure recoverable key compromise and robust freshness guarantees.

Under the hood, Docker Content Trust handles name resolution from IMAGE tags to IMAGE digests by signing its own metadata -- when Content Trust is enabled, docker will verify the signatures and expiration dates in the metadata before rewriting a pull by tag command to a pull by digest.

In this step you will enable Docker Content Trust, sign images as you push them, and pull signed and unsigned images.

1.  Enable Docker Content Trust by setting the DOCKER_CONTENT_TRUST environment variable.

   ```
   $ export DOCKER_CONTENT_TRUST=1
   ```

   > **Note:** If you are using `sudo` with your Docker commands, you will need to preceded the above command so that it looks like this`sudo export DOCKER_CONTENT_TRUST=1`

   It is worth noting that although Docker Content Trust is now enabled, all Docker commands remain the same. Docker Content Trust will work silently in the background.

2. Pull the `alpine:latest` signed image.

   ```
   $ docker rmi -f alpine:latest
   $ docker pull alpine:latest

		   Pull (1 of 1): alpine:latest@sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c4367d
		sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c4367d: Pulling from library/alpine
		c9b1b535fdd9: Pull complete 
		Digest: sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c...
   ```

   Look closely at the output of the `docker pull` command and take particular notice of the name translation - how the command is translated to the digest as shown below:

   ```
		   Pull (1 of 1): alpine:latest@sha256:ab00606a42621fb68f2ed6ad3c88be54397f981a7b70a79db3d1172b11c4367d
   ```

3.  Pull and unsigned image.

   ```
   $ docker rmi -f tenstartups/alpine
   $ docker pull tenstartups/alpine

   Using default tag: latest
Error: remote trust data does not exist for docker.io/tenstartups/alpine: notary.docker.io does not have trust data for docker.io/tenstartups/alpine
   ```

   You cannot pull unsigned images with Docker Content Trust enabled. Once Docker Content Trust is enabled you can only pull, run, or build with trusted images.


# Step 4: Official images

All images in Docker Hub under the `library` organization (currently viewable at: https://hub.docker.com/explore/)
are deemed "Official Images."  These images undergo a rigorous, [open-source](https://github.com/docker-library/official-images/)
review process to ensure they follow best practices. These best practices include signing, being lean, and having clearly written Dockerfiles. For these reasons, it is strongly recommended that you use official images whenever possible.

Official images can be pulled with just their name and tag. You do not have to precede the image name with `library/` or any other repository name.


# Summary

Congratulations. You now know the advantages of image digests over image tags. You have also seen how to enable Docker Content Trust and push signed images. Finally you have seen how to perform some advanced tasks using the notary server and client.