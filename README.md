# Values in ACR Tasks 

You can use an ACR task to pass in a values file which can be consumed in parts of your build pipeline. 

## Using an ARG

For this we are using the following files.

```bash
.
├── Dockerfile
├── acb.yaml
└── values.yaml

```

The `Dockerfile`  outputs a `build-arg`.

```Dockerfile
FROM bash
ARG test_arg
RUN echo "ARG_VALUE=$test_arg"
```

You can perform a local build to validate this 

```bash
$ docker build --build-arg test_arg=passed_in_value -t test .
Sending build context to Docker daemon  4.096kB
Step 1/3 : FROM bash
 ---> a88d58e960ac
Step 2/3 : ARG test_arg
 ---> Running in 9f3a06c14611
Removing intermediate container 9f3a06c14611
 ---> 862f8bd03c06
Step 3/3 : RUN echo "ARG_VALUE=$test_arg"
 ---> Running in 54ad539af0ab
ARG_VALUE=passed_in_value
Removing intermediate container 54ad539af0ab
 ---> 1aff2d42ed91
Successfully built 1aff2d42ed91
Successfully tagged test:latest
```

When using ACR task you can create a `values.yaml` file that can be passed into the task. 

```yaml
TEST_ARG : "TEST_VALUE"
```

Finally an `acb.yaml` that builds the image and pushes it into the registry. 

```yaml
steps:
  - build: --build-arg test_arg={{.Values.TEST_ARG}} -f Dockerfile -t {{.Run.Registry}}/test:values .
  - push: 
    - {{.Run.Registry}}/test:values

```

## Runing the Task

You can use the az cli to build and push the image as follows. 

```
az configure --defaults acr=myregistry
az acr run -f acb.yaml --value values.yaml .
```

