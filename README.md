# About

**General Docker image for executing headless Google Chrome or Firefox Protractor e2e test cases in a Docker container. The created image does not contain any test code or project. This is the environment for running Protractor test cases. So this image is test project independent.**

This project was forked from [SequenceIQ's docker-e2e-protractor project](https://github.com/sequenceiq/docker-e2e-protractor). The reason for the divergence is that SequenceIQ's script will only execute the Protractor binary. However, projects may want a higher order script to do more than just run tests, such as use the [protractor-webpack module](https://github.com/thompsnm/protractor-webpack) to synchronize serving a development build of your service with running Protractor tests. This project allows for executing arbitrary scripts instead of always calling to the Protractor binary.

# To run your test cases in this image

1. Pull the `hortonworks/docker-e2e-protractor` image from [DockerHub](https://hub.docker.com/r/hortonworks/docker-e2e-protractor/)
2. If you have any environment variable which is used for your test project, provide here [environment file](utils/testenv).
3. **The Protractor configuration file is vital for the Docker image**. Add your e2e test configuration JS file (for example `e2e.conf.js`). Beside this you can provide additional parameters here for protractor.
4. You can see some example for execute your protractor tests in this [Docker](https://docs.docker.com/engine/installation/) container:
    ```
    docker run -it --rm --name protractor-runner -v $(PWD):/protractor/project hortonworks/docker-e2e-protractor protractor e2e.conf.js    
    docker run -it --rm --name protractor-runner --env-file utils/testenv -v $(PWD):/protractor/project hortonworks/docker-e2e-protractor protractor e2e.conf.js --suite smoke
    docker run -it --rm --name protractor-runner -e USERNAME=teszt.elek -e PASSWORD=Teszt12 -v $(PWD):/protractor/project hortonworks/docker-e2e-protractor protractor e2e.conf.js --suite regression
    docker run -it --rm --name protractor-runner --privileged --net=host -v /dev/shm:/dev/shm -v $(PWD):/protractor/project hortonworks/docker-e2e-protractor protractor e2e.conf.js --suite smoke    
    ```

  - `utils/testenv` the location (full path) of the `testenv` file on your machine. This file can contain environment variables for your new container.
  - `USERNAME=teszt.ele` a single environment variable that is passed for the new container.
  - `$(PWD)` or `pwd` the root folder of your Protractor test project:
    - For example the local folder where the [ULUWATU functional E2E tests](https://github.com/sequenceiq/uluwatu-e2e-protractor) project has been cloned from GitHub.
    - The use of **PWD is optional**, you do not need to navigate to the Protractor test project root. If it is the case, you should add the full path of the root folder instead of the `$(PWD)`.
  - `e2e.conf.js --suite regression` in case of you defined [test suites](http://www.protractortest.org/#/page-objects#configuring-test-suites) in your Protractor configuration, you can add these here

# Advanced options and information

## Protractor direct connect
Protractor can test directly using Chrome Driver or Firefox Driver, [bypassing any Selenium Server](https://github.com/angular/protractor/blob/master/docs/server-setup.md#connecting-directly-to-browser-drivers). **The advantage of direct connect is that your test project start up and run faster.**

To use this, you should change your [config file](https://github.com/sequenceiq/uluwatu-e2e-protractor/blob/master/e2e.conf.js#L20):
```
directConnect: true
```
>**If this is true, settings for seleniumAddress and seleniumServerJar will be ignored.** If you attempt to use a browser other than Chrome or Firefox an error will be thrown.

## No sandbox for Google Chrome
Chrome does not support to [running it in container](https://github.com/travis-ci/travis-ci/issues/938#issuecomment-77785455). So you need to start the Chrome Driver with `--no-sandbox` argument to avoid errors.

In the [Protractor configuration file](https://github.com/sequenceiq/uluwatu-e2e-protractor/blob/master/e2e.conf.js#L25-L30):
```
capabilities: {
     'browserName': 'chrome',
     /**
      * Chrome is not allowed to create a SUID sandbox when running inside Docker
      */
     'chromeOptions': {
         'args': ['no-sandbox']
     }
},
```

## --privileged
Chrome uses sandboxing, therefore if you try and run Chrome within a non-privileged container you will receive the following message:

> "Failed to move to new namespace: PID namespaces supported, Network namespace supported, but failed: errno = Operation not permitted".

The `--privileged` flag gives the container almost the same privileges to the host machine resources as other processes running outside the container, which is required for the sandboxing to run smoothly.

<sub>Based on the [Webnicer project](https://hub.docker.com/r/webnicer/protractor-headless/).</sub>

## Makefile
We created a very simple [Makefile](https://github.com/sequenceiq/docker-e2e-protractor/blob/master/Makefile) to be able build and run easily our Docker image:
```
make build
```
then
```
make run
```
or you can run the above commands in one round:
```
make all
```
The rules are same as in case of [To run your test cases in this image](#to-run-your-test-cases-in-this-image).

## In-memory File System /dev/shm (Linux only)
Docker has hardcoded value of 64MB for `/dev/shm`. Error can be occurred, because of [page crash](https://bugs.chromium.org/p/chromedriver/issues/detail?id=1097) on memory intensive pages. The easiest way to mitigate the problem is share `/dev/shm` with the host.
```
docker run -it --rm --name protractor-runner --env-file utils/testenv -v /dev/shm:/dev/shm -v $(PWD):/protractor/project sequenceiq/protractor-runner
```
The size of `/dev/shm` in the Docker container can be changed when container is made with [option](https://github.com/docker/docker/issues/2606) `--shm-size`.

For Mac OSX users [this conversation](http://unix.stackexchange.com/questions/151984/how-do-you-move-files-into-the-in-memory-file-system-mounted-at-dev-shm) can be useful.

<sub>Based on the [Webnicer project](https://hub.docker.com/r/webnicer/protractor-headless/).</sub>

## --net=host
This options is required only if the dockerised Protractor is run against localhost on the host.

**Imagine this scenario:**
Run an http test server on your local machine, let's say on port 8000. You type in your browser http://localhost:8000 and everything goes smoothly. Then you want to run the dockerised Protractor against the same localhost:8000. If you don't use `--net=host` the container will receive the bridged interface and its own loopback and so the localhost within the container will refer to the container itself. Using `--net=host` you allow the container to share host's network stack and properly refer to the host when Protractor is run against localhost.

<sub>Based on the [Webnicer project](https://hub.docker.com/r/webnicer/protractor-headless/).</sub>
