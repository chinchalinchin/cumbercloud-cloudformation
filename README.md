# Cumberland Cloud Cloudformation

This repository houses the **CloudFormation** templates for provisioning the resources necessary for my personal website and portfolio. The templates in this repository will provision several **Lambda** functions, their event integrations (e.g. **APIGateway**, **CloudWatch**, etc.), a **Cloudfront** distribution and the **CodePipeline** CICD resources for continuously deploying changes to these components. The source code for the backend **Lambda** functions can be found [here](https://github.com/chinchalinchin/cumbercloud-lambdas). The source code for the frontend served through the **Cloudfront** distribution can be found [here](https://github.com/chinchalinchin/cumbercloud-web).

## Hard Dependencies

As much of the **AWS** configuration as possible is included in the **CloudFormation** templates. However, there are several resources that need provisioned manually,

1. [Route43 Domain name](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html)
2. [Amazon Certificate Manager SSL certificate](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
3. [Hosted Zone](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingHostedZone.html)

## Quickstart

Copy the *.sample.env* into a new *.env* and configure the values for your environment. See comments in file for more information on each variable. A helper script will read in the values from this value and use them to post the **Cloudformation** templates to **AWS**. If the script does not detect this file or the values are empty, it will prompt the user to input their values into the command line.

To provision a stack, execute the following script,

```shell
./scripts/provision-stack
```

The script will prompt the user to choose a stack to provision. The choices are: `web`, `lambda` and `cicd`. After the user selects a stack, the script will prompt the user to choose an action. The choices are `create`, `update` and `delete`. If this is the first deployment, the user should select `create`.

The `cicd` stack should be stood up before the other two stacks. This stack contains all the resources for continous integration and deployment, such as **CodeCommit** repos, **S3** buckets and **ECR** repos. In particular, the **ECR** repos must exist before the **Lambda** functions can be deployed, otherwise the stack deployment will fail. (This is the only hard dependency; technically the `web` stack can be stood up at anytime; the **Cloudfront** distribution will just be serving an empty **S3** bucket until the `cicd` stack can build and deploy the frontend app into the bucket.). 

Once the `cicd` stack is stood up, the pipelines will initially fail for two reasons: 

1. The **CodeCommit** repositories are empty. 
2. **CodePipeline** has no resources to deploy to.

To solve the second problem, stand up the `web` and `lambda` stack with `./scripts/provision-stack`.

To solve the first problem, clone the frontend and backend repositories found in the introduction of this *README*. 

```
git clone https://github.com/chinchalinchin/cumbercloud-web
git clone https://github.com/chinchalinchin/cumbercloud-lamdbas
```

Add the remote SSH urls for the **CodeCommit** repos (after you have [setup your CodeCommit SSH key](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html)) and then push them up to the new remote,

```
cd cumbercloud-web
git remote add codecommit <web-ssh-url>
git push
cd ../cumbercloud-lambdas
git remote add codecommit <lambdas-ssh-url>
git push
```

This will kick off the pipeline again. Assuming the `web` and `lambda` stack have been stood up, you should now have a fully functional web application and a serverless backend hooked into a **CodeCommit**-**CodeBuild**-**CodePipeline** CICD pipeline. Simply push new code to the **CodeCommit** repositories to redeploy.

## Linter

Install [cfn-lint](https://github.com/aws-cloudformation/cfn-lint),

```shell
pip install -r requirements.txt
```

And then invoke the helper script,

```shell
./scripts/lint-templates
```

## Notes

### ACM SSL Certificate

1. The certificate used to sign requests must be of the form `*.domain`