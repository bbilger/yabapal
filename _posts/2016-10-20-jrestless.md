---
title: JRestless - Build Serverless JAX-RS Applications
date: 2016-10-20T01:00:00+00:00
author: Björn Bilger
permalink: /2016/10/20/jrestless-build-serverless-jax-rs-applications/
excerpt: Use JRestless to run JAX-RS applications on AWS Lambda - including integration for Spring 4.x
categories:
  - Java
  - JRestless
tags:
  - Java
  - AWS Lambda
  - JRestless
  - JAX-RS
  - Jersey
---

AWS Lambda is a very interesting serverless computing service provided by Amazon. It allows you to run code without the burden of setting up a server - and even better you only pay what you use with the first 1,000,000 calls being free of charge.

The only downside is that you get locked in into that stack once you start developing applications with or rather for it.

That's why I am working on a framework called JRestless to avoid the cloud vendor lock-in.

JRestless is designed to use AWS Lambda (and AWS API Gateway) to run your RESTful APIs and/or microservices using JAX-RS. Jersey is used to provide compliance with the JAX-RS standard and allows you to use its extensions, including integration for **Spring 4.x**, as well.

You can find JRestless together with example applications and documentation on GitHub: [https://github.com/bbilger/jrestless](https://github.com/bbilger/jrestless)

Avoiding the cloud vendor lock-in by using JAX-RS allows you to develop, test and run your applications locally and - if required - move your application from AWS Lambda to another FaaS (Function as a Service) environment or a normal application server.

JRestless abstracts away AWS Lambda (and AWS API Gateway) specifics and provides a JAX-RS container allowing you to develop your application using JAX-RS. In case you are not familiar with JAX-RS: JAX-RS is a standard for Java to develop REST APIs.

JRestless doesn't implement a JAX-RS container itself but uses [Jersey](https://jersey.java.net/) - the reference implementation for JAX-RS. So you get a JAX-RS compliant container and you can use mostly all JAX-RS features like request/response filters, plus some of the extension Jersey provides on top which includes integration for **Spring 4.x** to name just one.

JRestless acts as an integration layer between FaaS environments and Jersey. Since AWS Lambda is the only Faas environment that supports Java at moment, it is the only supported environment right now. JRestless, however, is designed to support other platforms, as well, once Java is added.

You can use JRestless to develop two different types of AWS Lambda functions:

1. _Gateway functions_ that can be invoked through AWS API Gateway and so normal HTTP request.
2. _Service functions_ that can be invoked by other Lambda functions (or directly using the AWS SDK). The event object to invoke those function types is similar to a HTTP request. You can abstract the fact that you invoke a Lambda function away by using JRestless' feign client, allowing you to transparently call those function through feign. You can read more about those functions on Github: [jrestless-aws-service-handler](https://github.com/bbilger/jrestless/tree/master/aws/service/jrestless-aws-service-handler)

In the beginning it might be difficult to configure AWS properly and to deploy your Lambda functions. So you might be interested in the fact that you can use the [serverless framework](https://github.com/serverless/serverless) together with JRestless to get your applications deployed easily.

Since _gateway functions_ are probably the most interesting ones, I want to end this post with a simple example.

Install _serverless_ as described in the docs https://serverless.com/framework/docs/guide/installing-serverless/

Setup your AWS account as described in the docs https://serverless.com/framework/docs/providers/aws/setup/

Create a new function using _serverless_

```bash
mkdir aws-gateway-usage-example
cd aws-gateway-usage-example
serverless create --template aws-java-gradle --name aws-gateway-usage-example
rm -rf src/main/java/hello # remove the classes created by the template
mkdir -p src/main/java/com/jrestless/aws/examples # create the package structure
```
Replace `serverless.yml` with the following contents:

```yml
service: aws-gateway-usage-example-service

provider:
  name: aws
  runtime: java8
  stage: dev
  region: eu-central-1

package:
  artifact: build/distributions/aws-gateway-usage-example.zip

functions:
  sample:
    handler: com.jrestless.aws.examples.RequestHandler
    events:
      - http:
          path: sample/{proxy+}
          method: any

```

Replace `build.gradle` with the following contents:

```gradle
apply plugin: 'java'

repositories {
  jcenter()
  mavenCentral()
}
dependencies {
  compile(
    'com.jrestless.aws:jrestless-aws-gateway-handler:0.3.0',
    'org.glassfish.jersey.media:jersey-media-json-jackson:2.23'
  )
}
task buildZip(type: Zip) {
  archiveName = "${project.name}.zip"
  from compileJava
  from processResources
  into('lib') {
    from configurations.runtime
  }
}
build.dependsOn buildZip

```

Create a new JAX-RS resource and a JAXB DTO (`src/main/java/com/jrestless/aws/examples/SampleResource.java`):

```java
package com.jrestless.aws.examples;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@Path("/sample")
public class SampleResource {
  @GET
  @Path("/health")
  @Produces({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
  public Response getHealth() {
    return Response.ok(new HealthStatusDto("up and running")).build();
  }
  @XmlRootElement // for JAXB
  public static class HealthStatusDto {
    private String statusMessage;
    @SuppressWarnings("unused")
    private HealthStatusDto() {
      // for JAXB
    }
    HealthStatusDto(String statusMessage) {
      this.statusMessage = statusMessage;
    }
    @XmlElement // for JAXB
    public String getStatusMessage() {
      return statusMessage;
    }
  }
}
```

Create the request handler (`src/main/java/com/jrestless/aws/examples/RequestHandler.java`):
```java
package com.jrestless.aws.examples;

import com.jrestless.aws.gateway.GatewayResourceConfig;
import com.jrestless.aws.gateway.handler.GatewayRequestObjectHandler;

public class RequestHandler extends GatewayRequestObjectHandler {
  public RequestHandler() {
    init(new GatewayResourceConfig().packages("com.jrestless.aws.examples"));
    start();
  }
}
```

Build your function from within the directory `aws-gateway-usage-example`:
```bash
gradle build
```
This, amongst other things, creates a deployable zip file of your function (`build/distributions/aws-gateway-usage-example.zip`) using the dependent task `buildZip`.

Now you can deploy the function using _serverless_:

```bash
serverless deploy
```

If `serverless` is configured correctly, it should show you an endpoint in its output.
```
...
endpoints
  ANY - https://<SOMEID>.execute-api.eu-central-1.amazonaws.com/dev/sample/{proxy+}
...
```

Hit the endpoint:

```sh
curl -H 'Accept: application/json' 'https://<SOMEID>.execute-api.eu-central-1.amazonaws.com/dev/sample/health'
# {"statusMessage":"up and running"}
```

```sh
curl -H 'Accept: application/xml' 'https://<SOMEID>.execute-api.eu-central-1.amazonaws.com/dev/sample/health'
# <?xml version="1.0" encoding="UTF-8" standalone="yes"?><healthStatusDto><statusMessage>up and running</statusMessage></healthStatusDto>
```
