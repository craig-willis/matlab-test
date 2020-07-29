# Building Extensible MATLAB images with Docker

Repository to test MATLAB image building using pre-BuildKit and BuildKit approaches.

## Download installation media and license

* Create `/matlab` directory
* Download the MATLAB installation media in ISO format
* Convert ISO to `tar.gz` archive

```
ls /matlab
   R2019b_Linux.iso
   R2019b_Linux.tar.gz
   network.lic
```

## Serve installation media and license via local webserver

```
docker run -p 9090:80 -d -v /matlab:/usr/share/nginx/html nginx
```

### Build the base image

Build a base image containing all MATLAB dependencies, MATLAB itself,
and any other products selected in `matlab_installer_input_base.txt`.

```
docker build --network=host -t matlab-base -f Dockerfile.base . 
```

This takes ~10 minutes.

### Build user image

Build an image with user selected packages from `matlab_installer_input_user.txt`:

```
docker build --network=host -t matlab-user -f Dockerfile.user . 
```

Takes ~4 minutes for Econometrics toolbox only.


### Using BuildKit mode (experimental)

Docker 18.06 introduced the experimental [BuildKit mode](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/experimental.md. Among other things, this mode supports mounting volumes during build.

Enable experimental mode on Docker daemon:
```
{
  "experimental": true
}
```

Build an image that contains only the install media:

```
docker build --network=host -t matlab-install -f Dockerfile.install .
```

Build a user image that uses install via experimental mount option:

```
DOCKER_BUILDKIT=1 docker build --console=true --network=host --no-cache  -t matlab-mount -f Dockerfile.mount .
[+] Building 23.7s (10/10) FINISHED
 => local://dockerfile (Dockerfile.mount)                                                                                  0.0s
 => => transferring dockerfile: 103B                                                                                       0.0s
 => local://context (.dockerignore)                                                                                        0.1s
 => => transferring context: 2B                                                                                            0.0s
 => docker-image://docker.io/docker/dockerfile:experimental@sha256:de85b2f3a3e8a2f7fe48e8e84a65f6fdd5cd5183afa6412fff9caa  0.0s
 => => resolve docker.io/docker/dockerfile:experimental@sha256:de85b2f3a3e8a2f7fe48e8e84a65f6fdd5cd5183afa6412fff9caa6871  0.0s
 => => sha256:08fb2b2ca3d58e19d791e73dea16126df37608115532e748d63525688df457a5 897B / 897B                                 0.0s
 => => sha256:de85b2f3a3e8a2f7fe48e8e84a65f6fdd5cd5183afa6412fff9caa6871649c44 1.69kB / 1.69kB                             0.0s
 => => sha256:8c69d118cfcd040a222bea7f7d57c6156faa938cb61b47657cd65343babc3664 521B / 521B                                 0.0s
 => local://dockerfile                                                                                                     0.1s
 => => transferring dockerfile: 103B                                                                                       0.0s
 => local://context                                                                                                        0.1s
 => => transferring context: 2B                                                                                            0.0s
 => CACHED docker-image://docker.io/library/matlab-base:latest                                                             0.0s
 => /bin/sh -c wget -O /tmp/network.lic http://localhost:9090/network.lic &&     wget -O /matlab_installer_input_user.txt  0.8s
 => CACHED docker-image://docker.io/library/matlab-install:latest                                                          0.0s
 => /bin/sh -c cd /matlab-install &&  ./install -mode silent -inputFile /matlab_installer_input_user.txt -licensePath /t  17.0s
 => exporting to image                                                                                                     4.6s
 => => exporting layers                                                                                                    4.6s
 => => writing image sha256:ae881fd4b06d5571561d5d5ccf57d1741eea55cbf4df8e201b856adb4e1f408e                               0.0s
 => => naming to docker.io/library/matlab-mount
```

Much faster build time (< 1 minute) since we don't need to transfer the installer archive.

