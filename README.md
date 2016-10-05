## AWS Cleaner

[![Gem Version](https://badge.fury.io/rb/aws-cleaner.svg)](http://badge.fury.io/rb/aws-cleaner)
[![Dependency Status](https://gemnasium.com/badges/github.com/eheydrick/aws-cleaner.svg)](https://gemnasium.com/github.com/eheydrick/aws-cleaner)

AWS Cleaner listens for EC2 termination events produced by AWS [CloudWatch Events](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/WhatIsCloudWatchEvents.html)
and removes the instances from Chef and Sensu monitoring. Optionally
sends messages to Hipchat or Slack when actions occur.

### Prerequisites

You will need to create a CloudWatch Events rule that's configured to send termination event messages to SQS.

1. Create an SQS Queue for cloudwatch-events
2. Goto CloudWatch Events in the AWS Console
3. Click *Create rule*
4. Select event source of *EC2 instance state change notification*
5. Select specific state of *Terminated*
6. Add a target of *SQS Queue* and set queue to the cloudwatch-events queue created in step one
7. Give the rule a name/description and click *Create rule*

### Installation

1. `gem install aws-cleaner`

### Usage

```
Options:
  -c, --config=<s>    Path to config file (default: config.yml)
  -h, --help          Show this message
```

Copy the example config file ``config.yml.sample`` to ``config.yml``
and fill in the configuration details. You will need AWS Credentials
and are strongly encouraged to use an IAM user with access limited to
the AWS CloudWatch Events SQS queue.You will need to specify the region
in the config even if you are using IAM Credentials.

The app takes one arg '-c' that points at the config file. If -c is
omitted it will look for the config file in the current directory.

The app is started by running aws_config.rb and it will run until
terminated. A production install would start it with upstart or
similar.

### Webhooks

AWS Cleaner can optionally make an HTTP request to a specified endpoint. You can
also template the URL that is called. Templating is currently limited to a single
variable and the value can be either the Chef node name or the FQDN of the instance.

To enable webhooks, add a `:webhooks:` section to the config:

```
:webhooks:
  my-webhook:
    :url: 'http://my.webhook.com/blah/{fqdn}'
    :method: GET
    :template_variables:
      :variable: 'fqdn'
      :method: 'get_chef_fqdn' (or 'get_chef_node_name')
      :argument: '@instance_id'
```

Chat notifications can be sent when the webhook successfully executes. See
config.yml.sample for an example of the config.

### Limitations

- Currently only supports a single AWS region.
- Only support chef and sensu with non self signed certificates. Look at Aws Certificate Manager or Let's Encrypt for free SSL certificates.
