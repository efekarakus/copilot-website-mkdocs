List of all available properties for a `'Backend Service'` manifest.
```yaml
# Your service name will be used in naming your resources like log groups, ECS services, etc.
name: api

# Your service is reachable at "http://{{.Name}}.${COPILOT_SERVICE_DISCOVERY_ENDPOINT}:{{.Image.Port}}" but is not public.
type: Backend App

image:
  # Path to your service's Dockerfile.
  build: ./api/Dockerfile
  # Port exposed through your container to route traffic to it.
  port: 8080

  #Optional. Configuration for your container healthcheck.
  healthcheck:
    # The command the container runs to determine if it's healthy.
    command: ["CMD-SHELL", "curl -f http://localhost:8080 || exit 1"]
    interval: 10s     # Time period between healthchecks. Default is 10s if omitted.
    retries: 2        # Number of times to retry before container is deemed unhealthy. Default is 2 if omitted.
    timeout: 5s       # How long to wait before considering the healthcheck failed. Default is 5s if omitted.
    start_period: 0s  # Grace period within which to provide containers time to bootstrap before failed health checks count towards the maximum number of retries. Default is 0s if omitted.

# Number of CPU units for the task.
cpu: 256
# Amount of memory in MiB used by the task.
memory: 512
# Number of tasks that should be running in your service.
count: 1

variables:                    # Optional. Pass environment variables as key value pairs.
  LOG_LEVEL: info

secrets:                      # Optional. Pass secrets from AWS Systems Manager (SSM) Parameter Store.
  GITHUB_TOKEN: GITHUB_TOKEN  # The key is the name of the environment variable, the value is the name of the SSM      parameter.

# Optional. You can override any of the values defined above by environment.
environments:
  prod:
    count: 2               # Number of tasks to run for the "prod" environment.
```

* <a id="name" href="#name">`name`</a>: The name of your service.  
* `type`: The architecture type for your service. [Backend services](https://github.com/aws/copilot-cli/wiki/Service-Types#backend-service) is not reachable from the internet, but can be reached with service discovery from your other services.
* `image`: The image section contains parameters relating to the Docker build configuration and exposed port.
  * `build`: Can be specified either as a string or a map. If you specify a simple string, Copilot interprets it as the path to your Dockerfile. It will assume that the dirname of the string you specify should be the build context. The manifest:
    ```yaml
    image:
      build: path/to/dockerfile
    ```
    will result in the following call to docker build: `$ docker build --file path/to/dockerfile path/to` 

    You can also specify build as a map:
    ```yaml
    image:
      build:
       dockerfile: path/to/dockerfile
       context: context/dir
       args:
         key: value
    ```
    In this case, copilot will use the context directory you specified and convert the key-value pairs under args to --build-arg overrides. The equivalent docker build call will be: `$ docker build --file path/to/dockerfile --build-arg key=value context/dir`.

    You can also omit fields and Copilot will do its best to understand what you mean. For example, if you specify `context` but not `dockerfile`, Copilot will run Docker in the context directory and assume that your Dockerfile is named "Dockerfile." If you specify `dockerfile` but no `context`, Copilot assumes you want to run Docker in the directory that contains `dockerfile`.
 
    All paths are relative to your workspace root. 

  * `port`: The port exposed in your Dockerfile. Copilot should parse this value for you.
  * `healtcheck`: Optional configuration for container healthchecks.
    * `command`: The command to run to determine if the container is healthy.
    * `interval`: Time period between healthchecks in seconds. Default is 10s.
    * `retries`:  Number of times to retry before container is deemed unhealthy. Default is 2.
    * `timeout`: How long to wait before considering the healthcheck failed in seconds. Default is 5s.
    * `start_period`: Grace period within which to provide containers time to bootstrap before failed health checks count towards the maximum number of retries. Default is 0s.

* `cpu`: Number of CPU units for the task. See the [Amazon ECS docs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html) for valid CPU values.
* `memory`: Amount of memory in MiB used by the task. See the [Amazon ECS docs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html) for valid memory values.
* `count`: Can be specified either as a number or a map. If you specify a number:
  ```yaml
  count: 5
  ```
  The service will set the desired count to 5 and maintain 5 tasks in your service.

  Alternatively, you can specify a map for setting up autoscaling:
  ```yaml
  count:
    range: 1-10
    cpu_percentage: 70
    memory_percentage: 80
  ```
  * `range`: Specify a minimum and maximum bound for the number of tasks your service should maintain. 
  * `cpu_percentage`: Scale up or down based on the average CPU your service should maintain.
  * `memory_percentage`: Scale up or down based on the average memory your service should maintain.

* `variables`: Map of key-value pairs that represents environment variables that will be passed to your service. Copilot will include a number of environment variables by default for you.
* `secrets`: Map of key-value pairs that represents secret values from [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) that will passed to your service as environment variables securely. 
* `environments`: The environment section lets you overwrite any value in your manifest based on the environment you're in. In the example manifest above, we're overriding the count parameter so that we can run 2 copies of our service in our prod environment.