AWS Release Tasks
===============


Fabric release taks commands to use with AWS Beanstalk using boto.  Integrates with git by creating aliases for pushing code directly to the beanstalk.  Optional dependency is django-storages, package includes utilities for setting up your static and media backend for use in S3.


Dependencies
-----

* Fabric
* Django
* Boto
* prettytable
* git

Spatial Data Limitations
-----

The only fully supported db backend at this time is postgres / postgis using the psycopg2 driver. The problem is that MySQL does not support all spatial operations, see the [django docs](https://docs.djangoproject.com/en/1.7/ref/contrib/gis/db-api/#mysql-spatial-limitations).  In the future ebextensions will be added to support mysql without spatial support in the future.

The current AMI the tool works with is ami-35792c5c. The yum packages often change, this being a legacy AMI, packages do come and go and the only way to freeze this is to create a custom AMI will all prerequisites installed. I will create such AMI in the near future and provide the ID.  Right now the ebextensions file installs all required packages as an instance is built.

History
-----

The tool was inspired by  AWS Elastic Beanstalk command line tool, see [eb](https://github.com/radlws/AWS-ElasticBeanstalk-CLI).

Installation
------------------

Ensure the version of git you are using includes the points-at command, git 1.8+. See [Git Install](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

### Install using pip

    [sudo] pip install git+https://github.com/radlws/django-awseb-tasks.git

### Alternatively, install somewhere on your PYTHONPATH i.e. in  ../lib

This method will allow you to keep the module part of your repository.

    cd ../lib
    pip install --target . -U git+https://github.com/radlws/django-awseb-tasks.git

### Reference it in your fabfile.py

First set the required environment variables in your fab file, then import the tasks

    import os
    os.environ['PROJECT_NAME'] = os.getcwd().split('/')[-1]  # Import before aws_tasks, as it is used there.
    os.environ['DEFAULT_REGION'] = 'us-east-1'
    os.environ['DB_HOST'] = 'prod.your-db-url.us-east-1.rds.amazonaws.com'  # RDS DB URL, update accordingly']
    from aws_tasks import tasks as aws


### Sample Fabfile.py

    import os
    from fabric.api import task, local
    #env.project_name = os.getcwd().split('/')[-1]
    os.environ['PROJECT_NAME'] = os.getcwd().split('/')[-1]  # Import before aws_tasks, as it is used there.
    os.environ['DEFAULT_REGION'] = 'us-east-1'
    os.environ['DB_HOST'] = 'prod.czxygluip2xt.us-east-1.rds.amazonaws.com'  # RDS DB URL, update accordingly']
    from awseb_tasks import tasks as aws


Usage
-----

### Available commands

* list_environments  - Shows all available environments
* status - runs list_environments
* instances - Returns SSH connection string to available instance
* leader - Returns ssh connection string to leader instance
* list_instances - Shows all instances for an environment
* deploy - Deploy a release to the specified AWS Elastic Beanstalk environment. Requires site name & tag (release)
* dump_bucket - Downloads an S3 Bucket, given the bucket name
* manage - Run a manage command remotely, need host that you can get from leader command. use appropriate cert
* sw_creds - switch credential files for boto and eb if they exist in your home dir. Quickly switch accounts i.e kct and baker
* eb_init - creats aws.push and aws.config commands used by deploy
* new_creds
* generate_app_config - Generates .ebextensions/app.config file based on PROJECT_NAME in root of project
* environment_status:<env-name> - returns the environment health and status


### S3 Storage Backend

(OPTIONAL) Using the S3 backend for media and static (requires storages install). Add this to your settings file:

    DEFAULT_FILE_STORAGE = 'aws_tasks.storage_backends.MediaS3Storage'
    STATICFILES_STORAGE = 'aws_tasks.storage_backends.StaticS3Storage'
    THUMBNAIL_DEFAULT_STORAGE = DEFAULT_FILE_STORAGE

 
Example Usage
------------------

Assuming tasks are imported as aws. You can deploy, migrate and collectstatic like this:

    fab aws.deploy aws.leader aws.manage:migrate aws.manage:collectstatic
    
The 'leader' task stores the leader instance in env.hosts, manage makes an ssh connection which requires you use the correct ssh private key used to start the instance to connect to it. Make sure the ssh key is added to ssh agent so it gets picked up:

    chmod 600 ~/.ssh/id_rsa_aws 
    ssh-add ~/.ssh/id_rsa_aws 


Assumptions
------------------

The project name is used as the name of the application in Elastic Beanstalk. The site i.e. live, staging, content, are created in EB as project-site.
