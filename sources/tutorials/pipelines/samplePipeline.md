page_title: Deploy a sample application using Shippable's continuous delivery pipelines
page_description: Quick start for getting up to speed with pipelines
page_keywords: getting started, formations, quick start, documentation, shippable

#Deploy your first Pipeline

This tutorial walks you through how to configure a demo pipeline using Shippable.

###Fork the demo projects to your account

1. <a href="https://github.com/shippableSamples/samplePipelinesDemo" target="_blank">samplePipelinesDemo</a> repository contains code for the demo
1. <a href="https://github.com/shippableSamples/samplePipelinesTest" target="_blank">samplePipelinesTest</a> contains the configuration for deploying samplePipelinesDemo application to a Test cluster on ECS

###Create necessary  integrations

You will need the following integrations:

1. GitHub integration: Follow instructions on the [GitHub integration page](../../integrations/scm/github/) for the sections:
    * **Adding an Account Integration**
    * **Use your integration in your Pipeline configuration**
2. Docker Hub integration: Follow instructions at on the [Docker Hub integration page](../../integrations/imageRegistries/dockerHub/) for the sections: **Adding an Account Integration**

3. An integration for the Container Service where your cluster is located. This example uses Amazon ECS. You can add an Amazon ECS integration by [following instructions here](../../integrations/containerServices/ecs/)

###Set up CI for the sample application

Before you get started with setting up your deployments, let's set up CI for your sample application.

1. Add the Docker Hub account integration you created to your Subscription containing the forked **samplepipelinesdemo**. To do this, go to your Subscription's **Settings** tab and click on **Integrations** in the sidebar menu. Click on **Add integration**, name your integration, and then in the dropdown, choose your integration.

1. Next, make the following changes to the shippable.yml at the root of your forked sample application:
    * replace `shipdeploy` with the **Docker Hub** integration you created. Please use the integration name from your Subscription Settings here.
    * replace `shippablesamples` in the IMAGE_NAME with your Docker Hub org name where you want to push the image.

1. Enable your forked sample application for CI. Click on **Enable project** in the left sidebar menu, find the samplePipelinesDemo and click on **Enable**.

1. You can trigger a build for your sample application in one of two ways:
    * Go to the Shippable UI and click on `Build` for the project on the Subscription page
    * Make a small change to the project. In the /static/css/app.cat.css file, change the color value in the following code

```
.shippableText{
  color: #fff;
  text-align: center;
  font-size: 8em;
}
```
This should trigger a build on Shippable. Wait for your green build!

1. After the build is complete, check your Docker Hub org to ensure that the image was pushed successfully. Write down the tag of the image pushed (should be master.build_number if everything went well).

###Create a cluster
Cluster creation is not covered in this sample, since it assumes a cluster is already available. Create a cluster on your container service with at least one machine. There are no other constraints. Note down your cluster name and region.

###Edit pipeline configuration ymls

1. Open up **samplePipelinesTest/shippable.resources.yml** and make the following edits:
    * dv-img resource
        * replace `integration: dh-manishas` for the dv-img resource with `integration: <your docker hub integration name>`
        * replace `sourceName: shippableSamples/samplepipelinesdemo` with `sourceName: <your Docker Hub org name>/samplepipelinesdemo`
        * replace `versionName: master.1` with `version: <image tag you copied from Docker Hub>`
    * env-test-ecs resource
        * replace 'integration demo-manishas-ecs' for the env-test-ecs resource with `integration: <your ecs integration name>`
        * replace `demo-shippable-ecs-test` with your cluster name
        * replace `us-east-1` with the region where your cluster is located

### Understand the configuration

Before you proceed with setting up this pipeline on Shippable, let's take a moment to understand what it does.

The resources configured in shippable.resources.yml are:

1. dv-img is an [image](../../pipelines/resources/image/) resource for the image to be deployed.
1. dv-img-opts is a [dockerOptions](../../pipelines/resources/image/) resource which specifies options for the container, like memory, port mappings, etc.
1. env-test-ecs is a [cluster](../../pipelines/resources/cluster/) resource specifying where the demo application should be deployed

The jobs configured in shippable.jobs.yml are:

1. dv-man is a [manifest](../../pipelines/jobs/manifest/) job that creates a new service manifest each time the image dv-img is updated.
1. dv-test-ecs is a [deploy](../../pipelines/jobs/deploy/) job that deploys the manifest dv-man to the Test cluster env-test-ecs

### Seed your pipeline in Shippable

1. From the Shippable dashboard, go to the Subscription where you forked both repositories
1. Follow instructions on the Pipelines page to [seed your pipeline](../../pipelines/gettingStarted/#seedPipeline).
1. Go to the SPOG pill menu of your Pipelines tab and voila! You should see your pipeline there:

<img src="../../images/pipelines/seedSamplePipeline.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>

###Deploy to Test
Right click on the **dv-man** job in the SPOG view and click on `Run`. This will run the manifest job which creates a new service manifest. The deploy job is set up to run after manifest finishes, so it will be automatically triggered.

<img src="../../images/pipelines/samplePipelineTestDeploy.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>

###Check out your deployed demo on ECS

You can now go to your AWS management console and navigate to the deployed application by following the steps below:

1. Go to your ECS cluster by navigating to EC2 Container Service and clicking on the cluster name
1. Click on the Service Name starting with dv. This will take you to the Tasks page
1. Click on the Task to navigate to the containers page.
1. Expand the container to view the IP address of your deployed application
1. Click on the IP address will open up a new browser tab and show you the running application.

<img src="../../images/pipelines/demoApplication.png" alt="Shippable Continuous Integration and Delivery" style="width:800px;"/>

###Connect CI and Pipelines

Now that you have your pipeline up and running, you should connect it to your CI. On completing this step, every code change to your sample application will trigger a deployment to the Test cluster we set up in the previous steps.

To do this:

1. Create an API token for your account. Please note that only accounts with at least one paid subscription are allowed to create API tokens at this time. To do this, go to your **Account Settings** by clicking on the gear icon in the top navbar. Then click on **API tokens** in the left sidebar menu and create a token. Copy the token since you won't be able to see it again.  

1. Next, we will create an account integration of type 'Event Trigger'
    * Go to  **Integrations** in the left sidebar menu and then click on **Add Integration**
    * Select **Event Trigger** from the dropdown for **Master Integration** and complete the settings as shown below. Please make sure you update the `Authorization` textbox in the format `apiToken <token-value>`.
    <br>
<img src="../../images/pipelines/samplePipelineEventTrigger.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>

1. Add the integrations to your Subscription containing the forked samplePipelinesDemo. To do this, go to your Subscription's **Settings** tab and click on **Integrations** in the sidebar menu. Click on **Add integration**, name your integration, and then in the dropdown, choose the integration you created in the previous step.

1. Next, make the following changes to the shippable.yml at the root of your forked sample application:
    * Uncomment the `notifications` section
    * replace `triggerPipelinesDemo` with the name of the **Event Trigger** integration you created. Please use the integration name from your Subscription Settings here.

<img src="../../images/pipelines/samplePipelineConnectCI.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>

Try pushing a color change to your sample demo by going to the /static/css/app.cat.css file and changing color of .shippableText, and see your pipeline light up! Check your running application to see the new color.

###Adding a release step
Now that you have CI hooked up, you have a workflow from source control to a test environment. Next, we will hook up a release job that will be triggered manually when you are satisfied with your tests.

This release job tags your service with a version number starting with a seed version specified in your resources yml.

* Add the following to your shippable.resources.yml in the samplePipelinesTest repository:

```
  - name: dv-ver
    type: version
    seed:
      versionName: "1.0.0"
```
This creates a [version resource](../../pipelines/resources/version/) for your sample project.

* Add the following to your shippable.jobs.yml in the samplePipelinesTest repository:

```
  - name: dv-rel
    type: release
    steps:
      - IN: dv-ver
        switch: off
      - IN: dv-test-ecs
        switch: off
      - IN: dv-trigger
        switch: off
      - TASK: managed
        bump: minor
```
This creates a [release job](../../pipelines/jobs/release/) for your sample project. It takes the version resource we just created as an input. We are also adding a [trigger resource](../../pipelines/triggers/) that allows us to manually trigger the pipeline. This is defined in `shippable.triggers.yml`. This job is sequenced after the deployment job in your pipeline, and every time a release is created, it currently bumps up the minor version.

Commit your changes. You'll see the release job added at the end:

<img src="../../images/pipelines/samplePipelineAddRelease.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>


###Adding a production deployment

Each time a release is created, we will hook it up to deploy to production. You will need an existing cluster on your Container Service to set this up. To configure this:

* Add the following to your shippable.jobs.yml in the samplePipelinesTest repository:

```
  - name: dv-prod-ecs
    type: deploy
    steps:
      - IN: dv-rel
        switch: off
      - IN: env-prod-ecs
      - TASK: managed
```

* Add the following to your shippable.resources.yml in the samplePipelinesTest repository:

```
  - name: env-prod-ecs
    type: cluster
    integration: demo-manishas-ecs
    pointer:
      sourceName : "demo-shippable-ecs-prod"
      region: "us-east-1"
```
Make the following changes:

* Replace `demo-manisha-ecs` with the Container Service integration name from your subscription.
* Replace `demo-shippable-ecs-prod` with your prod cluster name.
* Replace `us-east-1` with the region where your cluster is located

This is similar to the job that deploys to test environment, but it is configured with the prod cluster and is sequenced after the releae job in your pipeline. The job as configured will not be triggered automatically when release is updated. You can turn `switch: on` to run it automatically each time a new release is created, or you can add a trigger resource like we did for the release job so you can trigger it with a commit.

<img src="../../images/pipelines/samplePipelineAddProd.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>

###Create a release and trigger deployment!

To create a new release, go to your shippable.triggers.yml and increment the counter by 1. This will trigger the release job and create a release with version 1.1.0.

Next, go to your Shippable UI SPOG view, right click on the production deployment job dv-prod-ecs and click on `Run`. This should deploy your application to your production cluster.

Go to your AWS Management Console and check out your application!

<img src="../../images/pipelines/samplePipelineProdDeploy.png" alt="Shippable Continuous Integration and Delivery" style="width:1000px;"/>
