---
layout: post
title: "Brew install all available AWS tools"
date: 2013-03-29 03:53:00 -0600
comments: true
categories: 
alias: /2013/03/29/brew-install-all-available-aws-tools/
---
If you happen to find yourself using a decent number of AWS services, you might find this useful. If there are any errata, please let me know.

As of this writing, this should install all of the formulas that inherit from `AmazonWebServicesFormula`:
```
brew install auto-scaling aws-cfn-tools aws-elasticache aws-elasticbeanstalk aws-iam-tools aws-sns-cli cloud-watch ec2-ami-tools ec2-api-tools elb-tools rds-command-line-tools
```

You'll get some errors about failing to link service, which is the same file for all of the formulas which try to link it. You'll have to force the link, but let's make sure the files are all the same.
<!-- more -->

```
clay@laptop:1.9.3: ~ $ which service
/usr/local/bin/service

clay@laptop:1.9.3: ~ $ ls -alht /usr/local/bin/service
lrwxr-xr-x  1 clay  admin    43B Mar 29 04:11 /usr/local/bin/service -> ../Cellar/auto-scaling/1.0.61.1/bin/service

clay@laptop:1.9.3: ~ $ shasum /usr/local/bin/service
2b8c2f25d30db009b04fc2b7ae27591d07c07d7a  /usr/local/bin/service

clay@laptop:1.9.3: ~ $ shasum /usr/local/Cellar/aws-cfn-tools/1.0.12/bin/service
2b8c2f25d30db009b04fc2b7ae27591d07c07d7a  /usr/local/Cellar/aws-cfn-tools/1.0.12/bin/service

clay@laptop:1.9.3: ~ $ shasum /usr/local/Cellar/cloud-watch/1.0.13.4/bin/service
2b8c2f25d30db009b04fc2b7ae27591d07c07d7a  /usr/local/Cellar/cloud-watch/1.0.13.4/bin/service

clay@laptop:1.9.3: ~ $ shasum /usr/local/Cellar/elb-tools/1.0.17.0/bin/service
2b8c2f25d30db009b04fc2b7ae27591d07c07d7a  /usr/local/Cellar/elb-tools/1.0.17.0/bin/service
```

So, now that we see they're all the same, we can go on and force the links:
```
brew link --overwrite elb-tools cloud-watch aws-cfn-tools
```

You might be able to get a a more up-to-date list by grepping for the AmazonWebServicesFormula class in the Homebrew repository.
```
grep -irI AmazonWebServicesFormula ./* | awk '{print "brew install " $1}' | sed 's/.\/Library\/Formula\///g' | sed 's/.rb:class//g' | grep -ve 'Library'
```

That should give you something similar to:
```
brew install auto-scaling
brew install aws-cfn-tools
brew install aws-elasticache
brew install aws-elasticbeanstalk
brew install aws-iam-tools
brew install aws-sns-cli
brew install cloud-watch
brew install ec2-ami-tools
brew install ec2-api-tools
brew install elb-tools
brew install rds-command-line-tools
```

