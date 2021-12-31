# Fortune Teller
Fortune teller implemented via API and web cluster using AWS

## Serverless API
Using the [provided Python application code](fortune_handler/) as a starting point, I deployed an AWS Lambda-backed API behind an Application Load Balancer. I used SAM as my Infrastructure as Code tool so I can locally build the serverless application for my Lambda function before deploying.

Using the SAM CLI, I initialized the serverless application with the `sam init` command at the root of the directory. Then, I built the application inside a Lambda-like Docker container in preparation of deploying with the `sam build --use-container` command. Next, I ran the function locally with the `sam local invoke` command to check that the function runs well. Finally, I deployed to my AWS aacount with the `sam deploy` command, although I added the `--guided` flag on the first try to create the `samconfig.toml` file to save the parameters for future deployments.

I first created the networking essentials such as the VPC, subnets, IGW, NAT Gateway, and Route Tables. Afterwards, I added an application load balancer, its security group, listener, and target group. Lastly, I added the Lambda function, its IAM role, and perrmission for elasticloadbalancing to invoke the function.

To increase the application's observability, I added a log group, metric filter for memory used, metric filter for memory size, duration alarm, errors alarm, invocations alarm, and throttles alarm.

## Elastic Infrastructure
I deployed the [provided web site](frontend/) on an EC2-based web server, specifically an Auto Scaling Group. The web application was able to call the API previously. With the Auto Scaling Group, the web application can also scale elastically based on load (using metrics). To keep everything in one place, I added an auto scaling group, an instance profile, its role, its launch configuration (which includes the script to bootstrap the web server), scaling policies, CPU alarms, its security group, and its target group to the [SAM template `template.yaml`](fortune_handler/template.yaml).

I changed the listener so that it now forwards traffic to the web server's target group. As for the previously-created Lambda function, I added a listener rule so that the listener directs traffic to the function's target group when the path-pattern is `/fortune`.

## Running Implementations
After deploying the stack to the account, I verified on the console that the EC2 web server is fully initialized with both status checks passed. I then went to the load balancer and copied its DNS name so I can then go to its web page which successfully renders the front end of the fortune handler. If I then add `/fortune` to the end of the URL, then I see the text of the fortune.