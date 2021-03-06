# Amazon ECS Container Agent Configuration<a name="ecs-agent-config"></a>

The Amazon ECS container agent supports a number of configuration options, most of which should be set through environment variables\. The following environment variables are available, and all of them are optional\.

If your container instance was launched with the Amazon ECS\-optimized AMI, you can set these environment variables in the `/etc/ecs/ecs.config` file and then restart the agent\. You can also write these configuration variables to your container instances with Amazon EC2 user data at launch time\. For more information, see [Bootstrapping Container Instances with Amazon EC2 User Data](bootstrap_container_instance.md)\.

If you are manually starting the Amazon ECS container agent \(for non\-Amazon ECS\-optimized AMIs\), you can use these environment variables in the docker run command that you use to start the agent with the syntax `--env=VARIABLE_NAME=VARIABLE_VALUE`\. For sensitive information, such as authentication credentials for private repositories, you should store your agent environment variables in a file and pass them all at one time with the `--env-file path_to_env_file` option\.


+ [Available Parameters](#ecs-agent-availparam)
+ [Storing Container Instance Configuration in Amazon S3](#ecs-config-s3)

## Available Parameters<a name="ecs-agent-availparam"></a>

The following are the available environment keys:

`ECS_CLUSTER`  
Example Values: `MyCluster`  
Default Value: `default`  
The cluster that this agent should check into\. If this value is undefined, then the `default` cluster is assumed\. If the `default` cluster does not exist, the Amazon ECS container agent attempts to create it\. If a non\-`default` cluster is specified and it does not exist, registration fails\.

`ECS_RESERVED_PORTS`  
Example Values: `[22, 80, 5000, 8080]`  
Default Value: `[22, 2375, 2376, 51678]`  
An array of ports that should be marked as unavailable for scheduling on this container instance\.

`ECS_RESERVED_PORTS_UDP`  
Example Values: `[53, 123]`  
Default Value: `[]`  
An array of UDP ports that should be marked as unavailable for scheduling on this container instance\.

`ECS_ENGINE_AUTH_TYPE`  
Example Values: `dockercfg | docker`  
Default Value: Null  
Required for private registry authentication\. This is the type of authentication data in `ECS_ENGINE_AUTH_DATA`\. For more information, see [Authentication Formats](private-auth.md#docker-auth-formats)\.

`ECS_ENGINE_AUTH_DATA`  
Example Values:   

+ `ECS_ENGINE_AUTH_TYPE=dockercfg`: `{"https://index.docker.io/v1/":{"auth":"zq212MzEXAMPLE7o6T25Dk0i","email":"email@example.com"}}`

+ `ECS_ENGINE_AUTH_TYPE=docker`: `{"https://index.docker.io/v1/":{"username":"my_name","password":"my_password","email":"email@example.com"}}`
Default Value: Null  
Required for private registry authentication\. If `ECS_ENGINE_AUTH_TYPE=dockercfg`, then the `ECS_ENGINE_AUTH_DATA` value should be the contents of a Docker configuration file \(`~/.dockercfg` or `~/.docker/config.json`\) created by running docker login\. If `ECS_ENGINE_AUTH_TYPE=docker`, then the `ECS_ENGINE_AUTH_DATA` value should be a JSON representation of the registry server to authenticate against, as well as the authentication parameters required by that registry \(such as user name, password, and email address for that account\)\. For more information, see [Authentication Formats](private-auth.md#docker-auth-formats)\.

`AWS_DEFAULT_REGION`  
Example Values: `us-east-1`  
Default Value: Taken from EC2 instance metadata\.  
The region to be used in API requests as well as to infer the correct back end host\.

`AWS_ACCESS_KEY_ID`  
Example Values: `AKIAIOSFODNN7EXAMPLE`  
Default Value: Taken from EC2 instance metadata\.  
The [access key](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) used by the agent for all calls\.

`AWS_SECRET_ACCESS_KEY`  
Example Values: `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`  
Default Value: Taken from EC2 instance metadata\.  
The [secret key](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) used by the agent for all calls\.

`AWS_SESSION_TOKEN`  
The [session token](http://docs.aws.amazon.com/STS/latest/UsingSTS/Welcome.html) used for temporary credentials\.  
Default Value: Taken from EC2 instance metadata\.

`DOCKER_HOST`  
Example Values: `unix:///var/run/docker.sock`  
Default Value: `unix:///var/run/docker.sock`  
Used to create a connection to the Docker daemon; behaves similarly to the environment variable as used by the Docker client\.

`ECS_LOGLEVEL`  
Example Values: `crit | error | warn | info | debug`  
Default Value: `info`  
The level to log at on `stdout`\.

`ECS_LOGFILE`  
Example Values: `/ecs-agent.log`  
Default Value: Null  
The path to output full debugging information to\. If blank, no logs are recorded\. If this value is set, logs at the debug level \(regardless of `ECS_LOGLEVEL`\) are written to that file\.

`ECS_CHECKPOINT`  
Example Values: `true | false`  
Default Value: If `ECS_DATADIR` is explicitly set to a non\-empty value, then `ECS_CHECKPOINT` is set to `true`; otherwise, it is set to `false`\.  
Whether to save the checkpoint state to the location specified with `ECS_DATADIR`\.

`ECS_DATADIR`  
Example Values: `/data`  
Default Value: Null  
The name of the persistent data directory on the container that is running the Amazon ECS container agent\. The directory is used to save information about the cluster and the agent state\.

`ECS_UPDATES_ENABLED`  
Example Values: `true | false`  
Default Value: `false`  
Whether to exit for ECS agent updates when they are requested\.

`ECS_UPDATE_DOWNLOAD_DIR`  
Example Values: `/cache`  
The filesystem location to place update tarballs within the container when they are downloaded\.

`ECS_DISABLE_METRICS`  
Example Values: `true | false`  
Default Value: `false`  
Whether to disable CloudWatch metrics for Amazon ECS\. If this value is set to `true`, CloudWatch metrics are not collected\.

`ECS_RESERVED_MEMORY`  
Example Values: 32  
Default Value: 0  
The amount of memory, in MiB, to reserve for processes that are not managed by ECS\.

`ECS_AVAILABLE_LOGGING_DRIVERS`  
Example Values: `["json-file","awslogs"]`  
Default Value: `["json-file","awslogs"]`  
The logging drivers available on the container instance\. The Amazon ECS container agent running on a container instance must register the logging drivers available on that instance with the `ECS_AVAILABLE_LOGGING_DRIVERS` environment variable before containers placed on that instance can use log configuration options for those drivers in tasks\. For information about how to use the `awslogs` log driver, see [Using the awslogs Log Driver](using_awslogs.md)\. For more information about the different log drivers available for your Docker version and how to configure them, see [Configure logging drivers](https://docs.docker.com/engine/admin/logging/overview/) in the Docker documentation\.

`ECS_DISABLE_PRIVILEGED`  
Example Values: `true | false`  
Default Value: `false`  
Whether launching privileged containers is disabled on the container instance\. If this value is set to `true`, privileged containers are not permitted\.

`ECS_SELINUX_CAPABLE`  
Example Values: `true | false`  
Default Value: `false`  
Whether SELinux is available on the container instance\.

`ECS_APPARMOR_CAPABLE`  
Example Values: `true | false`  
Default Value: `false`  
Whether AppArmor is available on the container instance\.

`ECS_ENGINE_TASK_CLEANUP_WAIT_DURATION`  
Example Values: `1h` \(Valid time units are "ns", "us" \(or "µs"\), "ms", "s", "m", and "h"\.\)  
Default Value: `3h`  
Time duration to wait from when a task is stopped until the Docker container is removed\. As this removes the Docker container data, be aware that if this value is set too low, you may not be able to inspect your stopped containers or view the logs before they are removed\. The minimum duration is `1m`; any value shorter than 1 minute is ignored\.

`ECS_CONTAINER_STOP_TIMEOUT`  
Example Values: `10m` \(Valid time units are "ns", "us" \(or "µs"\), "ms", "s", "m", and "h"\.\)  
Default Value: `30s`  
Time duration to wait from when a task is stopped before its containers are forcefully killed if they do not exit normally on their own\.

`HTTP_PROXY`  
Example Values: `10.0.0.131:3128`  
Default Value: Null  
The hostname \(or IP address\) and port number of an HTTP proxy to use for the ECS agent to connect to the internet \(for example, if your container instances do not have external network access through an Amazon VPC internet gateway or NAT gateway or instance\)\. If this variable is set, you must also set the `NO_PROXY` variable to filter EC2 instance metadata and Docker daemon traffic from the proxy\. For more information, see [HTTP Proxy Configuration](http_proxy_config.md)\.

`NO_PROXY`  
Example Values:   

+ Linux: `169.254.169.254,169.254.170.2,/var/run/docker.sock`

+ Windows: `169.254.169.254,169.254.170.2,\\.\pipe\docker_engine`
Default Value: Null  
The HTTP traffic that should not be forwarded to the specified `HTTP_PROXY`\. You must specify `169.254.169.254,/var/run/docker.sock` to filter EC2 instance metadata and Docker daemon traffic from the proxy\. For more information, see [HTTP Proxy Configuration](http_proxy_config.md)\.

`ECS_ENABLE_TASK_IAM_ROLE`  
Example Values: `true | false`  
Default Value: `false`  
Whether IAM roles for tasks should be enabled on the container instance for task containers with the `bridge` or `default` network modes\. For more information, see [IAM Roles for Tasks](task-iam-roles.md)\.

`ECS_ENABLE_TASK_IAM_ROLE_NETWORK_HOST`  
Example Values: `true | false`  
Default Value: `false`  
Whether IAM roles for tasks should be enabled on the container instance for task containers with the `host` network mode\. This variable is only supported on agent versions 1\.12\.0 and later\. For more information, see [IAM Roles for Tasks](task-iam-roles.md)\.

`ECS_DISABLE_IMAGE_CLEANUP`  
Example Values: `true`  
Default Value: `false`  
Whether to disable automated image cleanup for the Amazon ECS agent\. For more information, see [Automated Task and Image Cleanup](automated_image_cleanup.md)\.

`ECS_IMAGE_CLEANUP_INTERVAL`  
Example Values: `30m`  
Default Value: `30m`  
The time interval between automated image cleanup cycles\. If set to less than 10 minutes, the value is ignored\.

`ECS_IMAGE_MINIMUM_CLEANUP_AGE`  
Example Values: `30m`  
Default Value: `1h`  
The minimum time interval between when an image is pulled and when it can be considered for automated image cleanup\.

`ECS_NUM_IMAGES_DELETE_PER_CYCLE`  
Example Values: `5`  
Default Value: `5`  
The maximum number of images to delete in a single automated image cleanup cycle\. If set to less than 1, the value is ignored\.

`ECS_INSTANCE_ATTRIBUTES`  
Example Values: `{"custom attribute": "custom_attribute_value"}`  
Default Value: Null  
A list of custom attributes, in JSON form, to apply to your container instances\. Using this attribute at instance registration adds the custom attributes, allowing you to skip the manual method of adding custom attributes via the AWS Management Console\.  
Attributes added will not apply to container instances that are already registered\. To add custom attributes to already registered container instances, see [Adding an Attribute](task-placement-constraints.md#add-attribute)\.
For information about custom attributes to use, see [Attributes](task-placement-constraints.md#attributes)\.  
An invalid JSON value for this variable causes the agent to exit with a code of `5`\. A message appears in the agent logs\. If the JSON value is valid but there is an issue detected when validating the attribute \(for example if the value is too long or contains invalid characters\), then the container instance registration happens but the agent exits with a code of `5` and a message is written to the agent logs\. For information about how to locate the agent logs, see [Amazon ECS Container Agent Log](logs.md#agent-logs)\.

`ECS_ENABLE_CONTAINER_METADATA`  
Example Values: `true`  
Default Value: `false`  
When `true`, the agent creates a file describing the container's metadata\. The file can be located and consumed by using the container environment variable `$ECS_CONTAINER_METADATA_FILE`\.

`ECS_HOST_DATA_DIR`  
Example Values: `/var/lib/ecs`  
Default Value: `/var/lib/ecs`  
The source directory on the host from which `ECS_DATADIR` is mounted\. We use this to determine the source mount path for container metadata files in the case the ECS agent is running as a container\. We do not use this value in Windows because the ECS agent does not run as a container\.

## Storing Container Instance Configuration in Amazon S3<a name="ecs-config-s3"></a>

Amazon ECS container agent configuration is controlled with the environment variables described above\. The Amazon ECS\-optimized AMI checks for these variables in `/etc/ecs/ecs.config` when the container agent starts and configures the agent accordingly\. Certain innocuous environment variables, such as `ECS_CLUSTER`, can be passed to the container instance at launch time through Amazon EC2 user data and written to this file without consequence\. However, other sensitive information, such as your AWS credentials or the `ECS_ENGINE_AUTH_DATA` variable, should never be passed to an instance in user data or written to `/etc/ecs/ecs.config` in a way that they would show up in a `.bash_history` file\.

Storing configuration information in a private bucket in Amazon S3 and granting read\-only access to your container instance IAM role is a secure and convenient way to allow container instance configuration at launch time\. You can store a copy of your `ecs.config` file in a private bucket, and then use Amazon EC2 user data to install the AWS CLI and copy your configuration information to `/etc/ecs/ecs.config` when the instance launches\.

**To allow Amazon S3 read\-only access for your container instance role**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**\. 

1. Choose the IAM role you use for your container instances \(this role is likely titled `ecsInstanceRole`\)\. For more information, see [Amazon ECS Container Instance IAM Role](instance_IAM_role.md)\.

1. Under **Managed Policies**, choose **Attach Policy**\.

1. On the **Attach Policy** page, for **Filter**, type `S3` to narrow the policy results\.

1. Select the box to the left of the **AmazonS3ReadOnlyAccess** policy and choose **Attach Policy**\.

**To store an `ecs.config` file in Amazon S3**

1. Create an `ecs.config` file with valid environment variables and values from [Amazon ECS Container Agent Configuration](#ecs-agent-config) using the following format\. This example configures private registry authentication\. For more information, see [Private Registry Authentication](private-auth.md)\.

   ```
   ECS_ENGINE_AUTH_TYPE=dockercfg
   ECS_ENGINE_AUTH_DATA={"https://index.docker.io/v1/":{"auth":"zq212MzEXAMPLE7o6T25Dk0i","email":"email@example.com"}}
   ```

1. To store your configuration file, create a private bucket in Amazon S3\. For more information, see [Create a Bucket](http://docs.aws.amazon.com/AmazonS3/latest/user-guide/CreatingaBucket.html) in the *Amazon Simple Storage Service Getting Started Guide*\. 

1. Upload the `ecs.config` file to your Amazon S3 bucket\. For more information, see [Add an Object to a Bucket](http://docs.aws.amazon.com/AmazonS3/latest/user-guide/PuttingAnObjectInABucket.html) in the *Amazon Simple Storage Service Getting Started Guide*\.

**To load an `ecs.config` file from Amazon S3 at launch**

1. Complete the above procedures in this section to allow read\-only Amazon S3 access to your container instances and store an `ecs.config` file in a private Amazon S3 bucket\.

1. Launch new container instances by following the steps in [Launching an Amazon ECS Container Instance](launch_container_instance.md)\. In [[ERROR] BAD/MISSING LINK TEXT](launch_container_instance.md#instance-launch-user-data-step), use the following example script that installs the AWS CLI and copies your configuration file to `/etc/ecs/ecs.config`\.

   ```
   #!/bin/bash
   yum install -y aws-cli
   aws s3 cp s3://your_bucket_name/ecs.config /etc/ecs/ecs.config
   ```