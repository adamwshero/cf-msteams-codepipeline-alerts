[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.7](https://img.shields.io/badge/python-3.7-green.svg)](https://www.python.org/downloads/release/python-370/)
![](https://img.shields.io/maintenance/yes/2020.svg)

# codepipeline-build-alerts
Cloudformation custom resource to manage MS Teams notifications for AWS CodePipeline events.

## Usage
1. Create a new MS Teams channel for build success notifications (optional)
2. Create a new MS Teams channel for build failure notifications
3. Add the 'Incoming Webhook' connector to your target channel(s)
  - Name: CodePipeline
  - Upload Image: [images/AWS-CodePipeline.png](images/AWS-CodePipeline.png)
  - Copy the connector url
4. Deploy the serverless application.
  - Parameters:
    - MsTeamsWebHookAllAlerts: connector url from step2
    - (optional) MsTeamsWebHookOnlyFailures: connector url from step2 for failure channel

## Requirements

* AWS CLI already configured with Administrator permission
* [Python 3 installed](https://www.python.org/downloads/)
* [Docker installed](https://www.docker.com/community-edition)

## Development

### Setup Your Environment

1. Create venv (only need to do this once)

    ```python3 -m venv .venv```

1.  Activate venv

    ```source .venv/bin/activate```

1. Install dev requirements

    ```pip install -r code/requirements.txt```

### Run Tests - PyTest
There are 2 sets of pytests:

- [unit](code/tests/unit): these are typical unit tests that test individual functions and mock all dependencies
- [integration](code/tests/integration): these test whole modules/classes and may depend on external resources

To execute just the unit tests:

```bash
python -m pytest -c code/pytest.ini code/tests/unit
```

To execute all tests:

```
export AWS_PROFILE=yourAccountAlias; python -m pytest -c code/pytest.ini code/tests
```

### Lint - PyLint

```
python -m pylint --rcfile code/.pylintrc code/*.py
python -m pylint --rcfile code/.pylintrc tests/**/*.py
```

## Debugging w/SAM CLI
See SAM docs [here](https://github.com/awslabs/aws-sam-cli/blob/develop/docs/usage.md#debugging-python-functions)
1. Build the package
```
sam build -t template.yaml -m code/requirements.txt
```
2. Set a breakpoint on files in the `.aws-sam/build/Lambda` directory.  If you set breakpoints in the `code/src` directory they won't be obeyed.  *Note* breakpoints won't be registered until after the lambda_debugger.setup_debugging() method completes.
3. Run the command through SAM local enabling debug
```
sam local invoke Lambda -t .aws-sam/build/template.yaml -e sample-events/create-event.json -n code/sample-events/pipeline-execution-change-event.json --profile yourAccountAlias -d 5890
```
4. Attach the debugger in vs code
Use `SAM CLI Python` profile

## Deploy the Resource (no pipeline)

Get a SAML token for the environment you wish to deploy to...

#### Build/package your resource

```
sam build -t ./template.yaml -m ./code/requirements.txt
```
```
sam package --template-file .aws-sam/build/template.yaml --output-template-file packaged.yaml --s3-bucket {bucketname}  --profile yourAccountAlias --s3-bucket yourBucketName --s3-prefix yourPrefixName
```
#### Deploy your resource

```
cfn deploy -t packaged.yaml -c ./config.json --profile yourAccountAlias
```

#### Invoking function locally using a local sample payload**
It's possible to run the lambda locally using sam local invoke

```bash
sam local invoke TeamNotificationFunction --event sample-events/pipeline-execution-change-event.json --env-vars sample-events/test-vars.json
```

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called sam logs. sam logs lets you fetch logs generated by your Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
sam logs -n TeamNotificationFunction --stack-name dev-codepipeline-notifications --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Destroy

In order to delete our Serverless Application recently deployed you can use the following AWS CLI Command:

```bash
cfn delete-stack -c yourStackName --profile yourAccountAlias
```
## Resources
* https://aws.amazon.com/premiumsupport/knowledge-center/sns-lambda-webhooks-chime-slack-teams/
* https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using
* https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook
* https://medium.com/@sebastian.phelps/aws-cloudwatch-alarms-on-microsoft-teams-9b5239e23b64

# Owner
Adam Shero<br>
cloudarmy.io@gmail.com
