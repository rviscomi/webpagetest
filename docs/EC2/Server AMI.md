There is now a server AMI for WebPagetest for quickly spinning up a private instance.

AMI: ami-fcfd6194

Region: us-east-1

When you launch the instance, make sure to allow HTTP traffic through your EC2 firewall configuration/security group.

Some of the features of the server AMI are:
* Pre-configured with locations for all of the EC2 regions
* Will automatically start and stop test agents in regions as necessary to run tests
    * Default config limits Agents to 1 instance at a time in each region
    * Test agents will be terminated if they have been running for close to an hourly increment (since they are billed hourly) and haven't had work in the last 15 minutes.
* Defaults to a headless instance requiring API keys and use through the API only
    * Can be overridden by specifying headless=0 in the user data configuration
* Can archive tests to S3 if a bucket is configured and provided through user data
* Automatically updates the server and test agent code to the latest (hourly)
* Any settings can be specified or overridden through user data when first launching your new instance
* A default API key can be provided through user data which will be configured as a no-limit API key
    * Additional keys will need to be added manually to /var/www/webpagetest/www/settings/keys.ini

Known issues:
* Instances are not started to render video.  The plan is to move video rendering to the server but that is not in place yet.
* The lag time for starting new test agents can be as long as 10 minutes.
* S3 archiving does not currently re-use the EC2 key that is used for starting test agents.

To use the AMI you need to provide an EC2 key and secret (at a minimum) through user data when the instance is started.  Any other settings provided through user data will override existing settings in settings.ini when you start your instance the first time. After you have started your instance, user data will no longer change the settings (e.g. on stop and restart of your instance). Be sure to remove your key and secret from the user data of your instance for security.

To change settings after launching your instance, ssh to the instance and manually edit the file:

```
  sudo vim /var/www/webpagetest/www/settings/settings.ini
```

then bounce nginx:

```
  sudo service nginx restart
```

Example user data:

```
ec2_key=AKIAJKP75OFSROV5GWEQ
ec2_secret=<secret for the key>
api_key=MyAPIKey
headless=0
```

The full list of settings that can be specified is in [settings.ini.sample](https://github.com/WPO-Foundation/webpagetest/blob/master/www/settings/settings.ini.sample)