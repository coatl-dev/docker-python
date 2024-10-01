# coatl.dev Python Docker images

[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/coatl-dev/docker-python/coatl.svg)](https://results.pre-commit.ci/latest/github/coatl-dev/docker-python/coatl)

## Supported tags and respective `Dockerfile` links

- [`2`, `2.7`, `2.7-bookworm`, `2.7.18`, `2.7.18-bookworm`]
- [`2-slim`, `2.7-slim`, `2.7-slim-bookworm`, `2.7.18-slim`, `2.7.18-slim-bookworm`]

## Supported architectures

- amd64
- arm64

## How to use this image

### Create a `Dockerfile` in your Python app project

```dockerfile
FROM coatldev/python:2

WORKDIR /usr/src/app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD [ "python", "./your-daemon-or-script.py" ]
```

### Run a single Python script

For many simple, single file projects, you may find it inconvenient to write a
complete `Dockerfile`. In such cases, you can run a Python script by using the
Python Docker image directly:

```sh
docker run -it --rm --name my-running-script -v "$PWD":/usr/src/myapp -w /usr/src/myapp coatldev/python:2 python your-daemon-or-script.py
```

## Image Variants

The python images come in many flavors, each designed for a specific use case.

### `python:<version>`

This is the defacto image. If you are unsure about what your needs are, you
probably want to use this one. It is designed to be used both as a throw away
container (mount your source code and start the container to start your app), as
well as the base to build other images off of.

Some of these tags may have names like bookworm or bullseye in them. These are
the suite code names for releases of Debian⁠

and indicate which release the image is based on. If your image needs to install
any additional packages beyond what comes with the image, you'll likely want to
specify one of these explicitly to minimize breakage when there are new releases
of Debian.

This tag is based off of `buildpack-deps`. `buildpack-deps` is designed for the
average user of Docker who has many images on their system. It, by design, has a
large number of extremely common Debian packages. This reduces the number of
packages that images that derive from it need to install, thus reducing the
overall size of all images on your system.

### `python:<version>-slim`

This image does not contain the common Debian packages contained in the default
tag and only contains the minimal Debian packages needed to run `python`. Unless
you are working in an environment where only the `python` image will be deployed
and you have space constraints, we highly recommend using the default image of
this repository.

When using this image `pip install` will work if a suitable built distribution
is available for the Python distribution package being installed. `pip install`
may fail when installing a Python distribution package from a source
distribution. This image does not contain the Debian packages required to
compile extension modules written in other languages. Possible solutions if a
`pip install` fails include:

- Use this image and install any required Debian packages before running
  `pip install`.
- Use the default image of this repository. The default image contains the most
  commonly required Debian packages. The majority of arbitrary `pip install`s
  should be successful without additional header/development Debian packages.

<!-- Links -->
[`2`, `2.7`, `2.7-bookworm`, `2.7.18`, `2.7.18-bookworm`]: https://github.com/coatl-dev/docker-python/blob/coatl/2.7/bookworm/Dockerfile
[`2-slim`, `2.7-slim`, `2.7-slim-bookworm`, `2.7.18-slim`, `2.7.18-slim-bookworm`]: https://github.com/coatl-dev/docker-python/blob/coatl/2.7/slim-bookworm/Dockerfile