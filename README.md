# Tapioca

**Tapioca** is a [slack](https://slack.com/) [app](https://api.slack.com/start), that will make conversation groups with random people inside your organization and suggest something to talk about!

## How to use it

You have two diffent ways to use. If you don't wanna customize nothing. Just follow the default paramters option. But if you want to customize some rules, go to the other option.

### With default parameters

Just enter on https://www.yuca.live/tapioca/ and press on **Add Tapioca to Slack** and follow instructions.
This will create a new channel named **tapioca-time**. Then you can invite people to join the channel.
**Tapioca** will take care of the rest, making groups with folks to talk and suggesting topics to them talk about.

### With Customization

You will need to follow some instructions to deploy the tapioca app in your own infrastructure
This option will require you to make a new [Slack App](https://api.slack.com/start) and
[Amazon Web Services](https://aws.amazon.com/) account to run for now.

From AWS you will use:

- S3 - Storage the auth token
- Lambda - Run the Oauth scritpt and draw script (to create random conversations)
- Api Gateway - Create a URL to trigger Oauth Script and receive the calback from slack install button

We suggest to deploy using [serverless framework](https://www.serverless.com/).

## Architecture

We have two Lambdas:

1. `oauthLambda.js` - This function will receive the Oauth hook from **slack** and save workspace token to be used later. As Trigger for this lambda you will need an API Gateway endpoint. You need to add this URL inside slack redirect URLs.

2. `tapiocaLambda.js` - This function will require some kind of scheduled run, we use a CloudWatch Events/EventBridge, that is a CRON to schedule when it should run.

## How to run

First of all install the serverless framework in your machine. [Click here](https://www.serverless.com/framework/docs/getting-started/) to go to installation instructions.

After install serverless, go to [slack apps](https://api.slack.com/apps) and create a new slack app. You will be redirected to a basic information about your app. Copy the `Client ID` and `Client Secret`.

Open `serverless.yml`, go to environment section and fill the empty keys with your own `Client ID` and `Client Secret`. If you want, change the group size, this variable represent how many people will be added in the conversation.

Use the [AWS Cron Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) to select the date and time. Don't forget, AWS work with UTC, so remember to calculate the time based in your current location.

And, probably you will need to change the s3BucketName variable (inside `serverless.yml`) because s3 buckets are global and the name must be unique.

Now, execute in your terminal:

```bash
serverless deploy --stage=production
```

Copy the Api Gateway URL from the output. You will need to put this URL on your slack app Redirect URL. Also, allow this permissions: `channels:manage, channels:read, groups:read, im:read, mpim:read, users:read, groups:write, chat:write, im:write, mpim:write` inside Bot Scope.

Then, create the empty file inside your bucket:

```bash
aws s3api put-object --bucket tapioca-slack --key slack-credentials --body slack-credentials
```

This file will be used to store the auth token from slack.

And Finally, install in your slack the tapioca. You need to go to `Manage Distribution` and click `Add to Slack`.

So at this point, you will see a new channel call `tapioca-time` in your slack :)

## License

This repo is a clone of [yuca-live/tapioca](https://github.com/yuca-live/tapioca). We change some things but use their code.

Please, check their [LICENSE](LICENSE.txt) file for more information.
