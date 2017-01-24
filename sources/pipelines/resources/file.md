page_title: Unified Pipeline Resources
page_description: List of supported resources
page_keywords: Deploy multi containers, microservices, Continuous Integration, Continuous Deployment, CI/CD, testing, automation, pipelines, docker, lxc

# file
This resource type is used to specify a file, that could be transferred to cluser of nodes, during deployment.

It can be given as input to `manifest` job. During the deployment, of the

You can define a Filters resource by adding it to `shippable.resources.yml`
```
resources:
  - name: <string>                              #required
    type: file                                  #required
    integration: <integration name>             #optional
    pointer:
      sourceName: <string>                      #required
    seed:
      versionName: <string>
```

* resource `name` should be an easy to remember text string. This will appear in the visualization of this resource in the SPOG view. It is also used to refer to this resource in the `shippable.jobs.yml`. If you have spaces in your name, you'll need to surround the value with quotes, however, as a best practice we recommend not including spaces in your names.

* `type` is always set to 'cluster'.

* `integration` should be the name of the Subscription Integration you create for you subscription that leverages the credentials you set up in Account Integrations to connect to the artifact registry of your choice. To learn how to create integrations for a specific Container Service, please select from the list below:
	* [JFrog Artifactory](../../integrations/artifactRegistries/jfrogArtifactory/)

* `sourceName` is the URI denoting that file. The value is different based on the integration used.

__No integration__: If there is no integration present, then `sourceName` could be a URL denoting the location of the file on the web.

__JFrog Artifactory__: The `sourceName` is of format `repositoryName/path` of a artifactory repository file. For example, the `index.html` shown in the below picture will have the `sourceName` as `pages/index.html`

<img src="/pipelines/images/resources/fileResourceJFrogArtifactory.png" alt="JFrog Artifactory" style="width:700px;"/>
