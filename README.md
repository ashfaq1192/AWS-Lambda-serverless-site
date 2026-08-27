# serverless-site

This project contains source code and supporting files for a serverless application that you can deploy with the SAM CLI. It includes the following files and folders:

* **`hello_world`** - Code for the application's Lambda function.
* **`events`** - Invocation events that you can use to invoke the function.
* **`tests`** - Unit tests for the application code.
* **`template.yaml`** - A template that defines the application's AWS resources.

The application uses several AWS resources, including Lambda functions and an API Gateway API. These resources are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates your application code.

## Beginner Guide: Replacing Cloud9
Cloud9 is not required for this project. The current workflow is VS Code, the AWS Toolkit, AWS CLI, and AWS SAM CLI. You can work locally or use VS Code Remote - SSH to edit files on this EC2 machine. SSH is only the development connection; Lambda runs in AWS after deployment.

### What we did in this session:
* Connected to the remote Linux server through VS Code.
* Confirmed that the AWS CLI was already installed and that AWS access came from the EC2 IAM role `MLOps-EC2-S3-Access`.
* Installed the AWS SAM CLI on the remote server.
* Created a Python 3.14 Hello World SAM application.
* Built the application successfully.
* Tried local invocation, which correctly reported that Docker or Finch was not installed.
* Started guided deployment, which stopped because the EC2 role lacked CloudFormation permissions.
* The application is built successfully, but it is not deployed until the IAM permission issue is fixed.

---

## 1. Install SAM CLI on the Remote Server
Run these commands in the VS Code terminal connected to the remote Linux server. The `x86_64` installer is appropriate for the x86 EC2 machine used in this session. *(Check your architecture with `uname -m` if unsure).*

```bash
cd ~
curl -L [https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip](https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip) -o aws-sam-cli-linux-x86_64.zip
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

## 2. Create the SAM Application

From the repository directory, create the application directory and start the interactive wizard:

```bash
cd ~/mlops-foundations
mkdir -p serverless-site
cd serverless-site
sam init

```

**The choices used in this session were:**

* **Template source:** AWS Quick Start Templates
* **Application template:** Hello World Example
* **Runtime and package type:** python3.14 and zip
* **X-Ray tracing:** enabled
* **CloudWatch Application Insights:** disabled
* **Structured JSON logging:** enabled
* **Project name:** serverless-site

*Note: The SAM wizard created `serverless-site/` inside the directory where it was run. That is why the final project path is `serverless-site/serverless-site`.*

SAM may also display a telemetry message during `sam init`. To disable SAM CLI telemetry for the current shell, run:

```bash
export SAM_CLI_TELEMETRY=0

```

## 3. Build the Application

Run SAM commands from the directory containing `template.yaml`. This project is nested one level deeper than its parent directory:

```bash
cd ~/mlops-foundations/serverless-site/serverless-site
sam build

```

This reads `template.yaml`, installs dependencies from `hello_world/requirements.txt`, and creates the build output in `.aws-sam/build`.

## 4. Test Locally

*Local testing requires Docker or Finch. Without a container runtime, skip this section and deploy directly to AWS.*

```bash
sam local invoke HelloWorldFunction --event events/event.json
sam local start-api

```

While `sam local start-api` is running, open another terminal and run:

```bash
curl [http://127.0.0.1:3000/hello](http://127.0.0.1:3000/hello)

```

## 5. Deploy to AWS

For the first deployment, run:

```bash
sam deploy --guided

```

**Suggested answers for this public Hello World sample:**

* **Stack Name:** serverless-site
* **AWS Region:** us-east-1
* **Confirm changes before deploy:** Y
* **Allow SAM CLI IAM role creation:** Y
* **Disable rollback:** N
* **Save arguments to samconfig.toml:** Y
* **HelloWorldFunction has no authentication:** Y

*Note: The unauthenticated answer makes the sample API publicly callable. For private data or production write operations, use IAM, Amazon Cognito, or an API authorizer instead.*

After the first guided deployment, you can simply use:

```bash
sam build
sam deploy

```

SAM prints an API Gateway URL after a successful deployment. Test the route with:

```bash
curl https://API_[ID.execute-api.us-east-1.amazonaws.com/Prod/hello/](https://ID.execute-api.us-east-1.amazonaws.com/Prod/hello/)

```

---

## Troubleshooting: CloudFormation Permission Error

If deployment fails with:

> `not authorized to perform: cloudformation:CreateChangeSet`

The issue is the IAM role, not the Lambda code. SAM deploys through CloudFormation, so the EC2 role must be allowed to create and update CloudFormation stacks and change sets. It also needs the required Lambda, API Gateway, S3, and IAM permissions; `iam:PassRole` is commonly required as well.

**Fix:** Ask an AWS administrator to update the EC2 role named `MLOps-EC2-S3-Access`. *(Do not put access keys in this repository)*. After the role is updated, run `sam deploy` again.

## Add a Static Website with S3

S3 stores the frontend files; Lambda does not host HTML, CSS, or JavaScript. Create a `frontend/` directory containing `index.html`, CSS, and JavaScript. The JavaScript can call the API Gateway URL:

```javascript
fetch("https://API_[ID.execute-api.us-east-1.amazonaws.com/Prod/hello/](https://ID.execute-api.us-east-1.amazonaws.com/Prod/hello/)")
  .then((response) => response.json())
  .then((data) => console.log(data));

```

Upload the files to a uniquely named bucket:

```bash
aws s3 mb s3://YOUR-UNIQUE-BUCKET-NAME --region us-east-1
aws s3 sync frontend/ s3://YOUR-UNIQUE-BUCKET-NAME/

```

*For a production website, keep the bucket private and put CloudFront in front of it using Origin Access Control and HTTPS. Do not make the bucket publicly writable.*

## Fetch, Tail, and Filter Lambda Logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`.

```bash
sam logs -n HelloWorldFunction --stack-name "serverless-site" --tail

```

## Tests

Tests are defined in the `tests` folder. Use PIP to install the test dependencies and run tests.

```bash
pip install -r tests/requirements.txt --user
# Unit test
python -m pytest tests/unit -v
# Integration test, requiring deploying the stack first.
AWS_SAM_STACK_NAME="serverless-site" python -m pytest tests/integration -v

```

## Cleanup

To delete the sample application that you created, use the AWS CLI:

```bash
sam delete --stack-name "serverless-site"

```

*Delete any separately created S3 objects and buckets as well. An S3 bucket must be empty before it can be deleted.*

