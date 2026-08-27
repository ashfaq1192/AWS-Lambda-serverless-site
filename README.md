# serverless-site

This project contains source code and supporting files for a serverless application that you can deploy with the SAM CLI. It includes the following files and folders.

- hello_world - Code for the application's Lambda function.
- events - Invocation events that you can use to invoke the function.
- tests - Unit tests for the application code.
- template.yaml - A template that defines the application's AWS resources.

The application uses several AWS resources, including Lambda functions and an API Gateway API. These resources are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates your application code.

If you prefer to use an integrated development environment (IDE) to build and test your application, you can use the AWS Toolkit.
The AWS Toolkit is an open source plug-in for popular IDEs that uses the SAM CLI to build and deploy serverless applications on AWS. The AWS Toolkit also adds a simplified step-through debugging experience for Lambda function code. See the following links to get started.

* [CLion](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [GoLand](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [IntelliJ](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [WebStorm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [Rider](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [PhpStorm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [PyCharm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [RubyMine](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [DataGrip](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/welcome.html)
* [Visual Studio](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/welcome.html)

## Deploy the sample application

The Serverless Application Model Command Line Interface (SAM CLI) is an extension of the AWS CLI that adds functionality for building and testing Lambda applications. It uses Docker to run your functions in an Amazon Linux environment that matches Lambda. It can also emulate your application's build environment and API.

To use the SAM CLI, you need the following tools.

* SAM CLI - [Install the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
* [Python 3 installed](https://www.python.org/downloads/)
* Docker - [Install Docker community edition](https://hub.docker.com/search/?type=edition&offering=community)

To build and deploy your application for the first time, run the following in your shell:

```bash
sam build --use-container
sam deploy --guided
```

The first command will build the source of your application. The second command will package and deploy your application to AWS, with a series of prompts:

* **Stack Name**: The name of the stack to deploy to CloudFormation. This should be unique to your account and region, and a good starting point would be something matching your project name.
* **AWS Region**: The AWS region you want to deploy your app to.
* **Confirm changes before deploy**: If set to yes, any change sets will be shown to you before execution for manual review. If set to no, the AWS SAM CLI will automatically deploy application changes.
* **Allow SAM CLI IAM role creation**: Many AWS SAM templates, including this example, create AWS IAM roles required for the AWS Lambda function(s) included to access AWS services. By default, these are scoped down to minimum required permissions. To deploy an AWS CloudFormation stack which creates or modifies IAM roles, the `CAPABILITY_IAM` value for `capabilities` must be provided. If permission isn't provided through this prompt, to deploy this example you must explicitly pass `--capabilities CAPABILITY_IAM` to the `sam deploy` command.
* **Save arguments to samconfig.toml**: If set to yes, your choices will be saved to a configuration file inside the project, so that in the future you can just re-run `sam deploy` without parameters to deploy changes to your application.

You can find your API Gateway Endpoint URL in the output values displayed after deployment.

## Use the SAM CLI to build and test locally

Build your application with the `sam build --use-container` command.

```bash
serverless-site$ sam build --use-container
```

The SAM CLI installs dependencies defined in `hello_world/requirements.txt`, creates a deployment package, and saves it in the `.aws-sam/build` folder.

Test a single function by invoking it directly with a test event. An event is a JSON document that represents the input that the function receives from the event source. Test events are included in the `events` folder in this project.

Run functions locally and invoke them with the `sam local invoke` command.

```bash
serverless-site$ sam local invoke HelloWorldFunction --event events/event.json
```

The SAM CLI can also emulate your application's API. Use the `sam local start-api` to run the API locally on port 3000.

```bash
serverless-site$ sam local start-api
serverless-site$ curl http://localhost:3000/
```

The SAM CLI reads the application template to determine the API's routes and the functions that they invoke. The `Events` property on each function's definition includes the route and method for each path.

```yaml
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

## Add a resource to your application
The application template uses AWS Serverless Application Model (AWS SAM) to define application resources. AWS SAM is an extension of AWS CloudFormation with a simpler syntax for configuring common serverless application resources such as functions, triggers, and APIs. For resources not included in [the SAM specification](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md), you can use standard [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) resource types.

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs generated by your deployed Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
serverless-site$ sam logs -n HelloWorldFunction --stack-name "serverless-site" --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Tests

Tests are defined in the `tests` folder in this project. Use PIP to install the test dependencies and run tests.

```bash
serverless-site$ pip install -r tests/requirements.txt --user
# unit test
serverless-site$ python -m pytest tests/unit -v
# integration test, requiring deploying the stack first.
# Create the env variable AWS_SAM_STACK_NAME with the name of the stack we are testing
serverless-site$ AWS_SAM_STACK_NAME="serverless-site" python -m pytest tests/integration -v
```

## Cleanup

To delete the sample application that you created, use the AWS CLI. Assuming you used your project name for the stack name, you can run the following:

```bash
sam delete --stack-name "serverless-site"
```

## Resources

For beginners, the following links provide an introduction to the AWS SAM specification, the SAM CLI, and serverless application concepts.
See the [AWS SAM developer guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) for an introduction to SAM specification, the SAM CLI, and serverless application concepts.

Next, you can use AWS Serverless Application Repository to deploy ready to use Apps that go beyond hello world samples and learn how authors developed their applications: [AWS Serverless Application Repository main page](https://aws.amazon.com/serverless/serverlessrepo/)

## Beginner guide: replacing Cloud9

Cloud9 is not required for this project. The current workflow is VS Code, the AWS Toolkit, AWS CLI, and AWS SAM CLI. You can work locally or use VS Code Remote - SSH to edit files on this EC2 machine. SSH is only the development connection; Lambda runs in AWS after deployment.

### What we did in this session

1. Connected to the remote Linux server through VS Code.
2. Confirmed that the AWS CLI was already installed and that AWS access came from the EC2 IAM role `MLOps-EC2-S3-Access`.
3. Installed the AWS SAM CLI on the remote server.
4. Created a Python 3.14 Hello World SAM application.
5. Built the application successfully.
6. Tried local invocation, which correctly reported that Docker or Finch was not installed.
7. Started guided deployment, which stopped because the EC2 role lacked CloudFormation permissions.

The application is built successfully, but it is not deployed until the IAM permission issue is fixed.

### Install SAM CLI on the remote server

Run these commands in the VS Code terminal connected to the remote Linux server. The `x86_64` installer is appropriate for the x86 EC2 machine used in this session. Check your architecture with `uname -m` if unsure.

```bash
cd ~
curl -L https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip \
  -o aws-sam-cli-linux-x86_64.zip
sudo apt-get update
sudo apt-get install -y unzip
unzip -q aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
sam --version
```

After confirming the installation, the temporary installer files can be removed:

```bash
rm -rf ~/aws-sam-cli-linux-x86_64.zip ~/sam-installation
```

### Create the SAM application

From the repository directory, create the application directory and start the interactive wizard:

```bash
cd ~/mlops-foundations
mkdir -p serverless-site
cd serverless-site
sam init
```

The choices used in this session were:

```text
Template source: AWS Quick Start Templates
Application template: Hello World Example
Runtime and package type: python3.14 and zip
X-Ray tracing: enabled
CloudWatch Application Insights: disabled
Structured JSON logging: enabled
Project name: serverless-site
```

The SAM wizard created `serverless-site/` inside the directory where it was run. That is why the final project path is `serverless-site/serverless-site`. Running `sam build` from the first directory caused the `template.yml not found` error; the correct template is `template.yaml` in the nested directory.

SAM may also display a telemetry message during `sam init`. To disable SAM CLI telemetry for the current shell, run:

```bash
export SAM_CLI_TELEMETRY=0
```

### Project directory

Run SAM commands from the directory containing `template.yaml`:

```bash
cd ~/mlops-foundations/serverless-site/serverless-site
```

This project is nested one level deeper than its parent directory:

```text
serverless-site/
└── serverless-site/
    ├── template.yaml
    ├── samconfig.toml
    ├── hello_world/app.py
    └── events/event.json
```

If you run `sam build` from the parent directory, SAM may report that `template.yml` cannot be found. Use the directory above, or specify the template explicitly:

```bash
sam build --template-file serverless-site/template.yaml
```

### Build

```bash
sam build
```

This reads `template.yaml`, installs dependencies from `hello_world/requirements.txt`, and creates the build output in `.aws-sam/build`.

### Test locally

Local testing requires Docker or Finch. Without a container runtime, skip this section and deploy directly to AWS.

```bash
sam local invoke HelloWorldFunction --event events/event.json
sam local start-api
```

While `sam local start-api` is running, open another terminal and run:

```bash
curl http://127.0.0.1:3000/hello
```

### Deploy

For the first deployment:

```bash
sam deploy --guided
```

Suggested answers for this public Hello World sample are:

```text
Stack Name: serverless-site
AWS Region: us-east-1
Confirm changes before deploy: Y
Allow SAM CLI IAM role creation: Y
Disable rollback: N
Save arguments to samconfig.toml: Y
HelloWorldFunction has no authentication: Y
```

The unauthenticated answer makes the sample API publicly callable. For private data or production write operations, use IAM, Amazon Cognito, or an API authorizer instead.

After the first guided deployment, use:

```bash
sam build
sam deploy
```

SAM prints an API Gateway URL after a successful deployment. Test the route with:

```bash
curl https://API_ID.execute-api.us-east-1.amazonaws.com/Prod/hello/
```

### Fixing the CloudFormation permission error

If deployment fails with:

```text
not authorized to perform: cloudformation:CreateChangeSet
```

the issue is the IAM role, not the Lambda code. SAM deploys through CloudFormation, so the EC2 role must be allowed to create and update CloudFormation stacks and change sets. It also needs the required Lambda, API Gateway, S3, and IAM permissions; `iam:PassRole` is commonly required as well.

Ask an AWS administrator to update the EC2 role named `MLOps-EC2-S3-Access`. Do not put access keys in this repository. After the role is updated, run:

```bash
sam deploy
```

For production, use a dedicated least-privilege deployment role rather than broad administrator permissions.

### Add a static website with S3

S3 stores the frontend files; Lambda does not host HTML, CSS, or JavaScript. Create a `frontend/` directory containing `index.html`, CSS, and JavaScript. The JavaScript can call the API Gateway URL:

```javascript
fetch("https://API_ID.execute-api.us-east-1.amazonaws.com/Prod/hello/")
  .then((response) => response.json())
  .then((data) => console.log(data));
```

Upload the files to a uniquely named bucket:

```bash
aws s3 mb s3://YOUR-UNIQUE-BUCKET-NAME --region us-east-1
aws s3 sync frontend/ s3://YOUR-UNIQUE-BUCKET-NAME/
```

For a production website, keep the bucket private and put CloudFront in front of it using Origin Access Control and HTTPS. Do not make the bucket publicly writable.

### Clean up

Remove the SAM stack when finished:

```bash
sam delete --stack-name serverless-site
```

Delete any separately created S3 objects and buckets as well. An S3 bucket must be empty before it can be deleted.
