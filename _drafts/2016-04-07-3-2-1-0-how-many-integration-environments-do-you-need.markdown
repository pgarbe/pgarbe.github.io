---
layout: post
title: "3, 2, 1, 0 - How many integration environments do you need?"
date: 2016-04-07 05:22:08 +0000
author: Philipp Garbe
comments: true
categories: [Docker, Continuous Delivery]
keywords: "Continuous Deployment, Continuous Delivery, Integration Environments"
description: "How many integration environments do you need? The less environments you have the faster your releases are."

---
How many integration environments do you have? What's the purpose of them? Do you really need them?
Do they enable you to deliver your applications continuously?


## The Pains
A couple of years ago, I was used to have 4 environments where my application had to be deployed. Over the years the maintenance of all of these environments became painful.

One problem was that they differ from production environment (some more, some less). The teams had to deal with bugs which were related to the environment but not with bugs of the application itself.

The other problem was that it took quite a while until your changes have been deployed from CI to Stable, Ref and finally Live (the names of our environments). It was frustrating to wait so long to see the benefits of your changes.

> These days you had to wait for several days until your commit was published to production. You might argue that this is quite fast. But believe me: You always want more. You get addicted to faster releases. Right now I'm complaining that a cycle time of 20 minutes (from commit to production) is quite long.

## The Cloud
A lot of things have been changed after we decided to [migrate to the cloud](http://inside.autoscout24.com/project_tatsu/2015/01/04/autoscout24-changes-technology-aws-linux-jvm/). We were able to establish [immutable servers](http://martinfowler.com/bliki/ImmutableServer.html) and [Configuration as Code](http://martinfowler.com/bliki/InfrastructureAsCode.html) as a default for our new services. This helped a lot to get rid of unnecessary environments. The teams now are dealing only with two environments, `dev` and `prod`.

But after while this also feels too cumbersome for me. Let me explain:

We realized that the truth is not in the code but in production. As we [compose our pages dynamically](http://inside.autoscout24.com/talks/2016/01/13/microservice-ui-composition/) you can run integration tests only in production, because you don't know which version of a dependent service is currently released and if it's the same version as on your staging environment.

![Different versions on different environments](/assets/staging_environments.png)

TODO: Explain pic?

Furthermore you've to monitor your service properly because other services can be released anytime and potentially break your service.

Integration in production leads to other challenges. How can you work on new features? How can you test the integration with other services before your users will be affected? The answer can fill another blog post but a lot of issues can be solved by `Feature Toggles`.

> There is a really good overview of Feature Toggles by Pete Hodgson [here](http://martinfowler.com/articles/feature-toggles.html).

As there are no integration tests on `dev` why should we keep it? Currently one reason is to test the deployment of your application. When you deal with [CloudFormation](https://aws.amazon.com/cloudformation/) these changes can't be tested locally, so it makes sense to have an environment where you can run this tests.

> I use the term `test` on purpose to ensure that this environment should only be used to *test* your deployments. It shouldn't be abused as an integration environment where you maybe check your latest hack before it goes to production.

## The Containers
When you think about `Docker` (or Containers in general) you might notice that even this `Dev` environment is not needed anymore. You can test your container during the build phase. And this container will be the same as the one which will be deployed to production. There is no benefit anymore to deploy it to an artificial `Dev` environment.

## The Learnings
What I've learned over the last years is that the faster you can deploy the better. It's a huge benefit for your business and an important USP against your competitors.

The less environments you have the faster your releases are. And it's a kind of maturity of your team how fast they can deliver.

What do you think?
