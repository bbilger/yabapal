---
id: 253
title: DPI for ServerEndpoints in Jetty and Java SE
date: 2015-04-22T23:57:41+00:00
author: Björn Bilger
layout: post
guid: https://bbilger.com/yabapal/?p=253
permalink: /2015/04/22/dpi-for-serverendpoints-in-jetty-and-java-se/
categories:
  - Java
tags:
  - Java SE
  - ServerEnpoint Instance
  - WebSocket
---
I am currently working on some small private Java SE project that uses WebSockets. What I wanted, was to inject an application-scoped instance into my ServerEnpoint:

<pre class="brush: java; title: ; notranslate" title="">@ServerEndpoint(value = "/")
public class ServerEndpoint {
	private ISomeDependentComponent someDependentComponent;
	public ServerEndpoint(ISomeDependentComponent someDependentComponent) {
		this.someDependentComponent = someDependentComponent;
	}
}
</pre>

I started using Tyrus. I was, however, a little disappointed when I realized that I cannot inject application-scoped instances into my ServerEndpoint. One can control the instantiation of a ServerEndpoint with a configurator like this, or use Weld for CDI in Java SE environment:

<!--more-->

<pre class="brush: java; title: ; notranslate" title="">@ServerEndpoint(value = "/", configurator=ServerEndpointConfigurator.class)
public class ServerEndpoint {...}
public class ServerEndpointConfigurator extends ServerEndpointConfig.Configurator {
 
    @Override
    public &lt;T&gt; T getEndpointInstance(Class&lt;T&gt; endpointClass) throws InstantiationException {
        return (T)new ServerEndpoint(new SomeDependentComponent());
    }
}
</pre>

But again, one does not have any control over the instantiation of the ServerEndpointConfigurator. So yes, one can inject something but only in case you know what instance to inject at compile time, or one introduces some other fancy mechanism like a static provider.

Since I lost hope in finding a proper solution for Tyrus and didn&#8217;t want to use Weld (total overkill), I decided to give Jetty a try.

After some time digging around in Jetty&#8217;s code, I finally found some solution.

We just need some ServerEndpointConfig:

<pre class="brush: java; title: ; notranslate" title="">public class ServerEndpointInstanceFactory&lt;T&gt; implements ServerEndpointConfig {
	
	private final Function&lt;Class&lt;T&gt;, T&gt; endpointFactory;
	private final Class&lt;T&gt; clazz;
	
	public ServerEndpointBaseConfigInstanceFactory(Class&lt;T&gt; clazz, Function&lt;Class&lt;T&gt;, T&gt; endpointFactory) {
		this.endpointFactory = endpointFactory;
		this.clazz = clazz;
	}
	@Override
	public Configurator getConfigurator() {
		return new Configurator() {
			@Override
	        public &lt;E&gt; E getEndpointInstance(Class&lt;E&gt; endpointClass) throws InstantiationException {
				return (E) endpointFactory.apply((Class&lt;T&gt;) endpointClass);
	        }
		};
	}
	@Override
	public Class&lt;?&gt; getEndpointClass() {
		return clazz;
	}
	//ommitted other methods; they just return null
}
</pre>

This ServerEndpointConfig will serve as a base config for the actual AnnotatedServerEndpointConfig selected by Jetty.[0] The AnnotatedServerEndpointConfig will pick the values either from our base config or the annotated ServerEndpoint.

Since we are now able to provide an **instance** of an additional ServerEndpointConfig. We are now able to indirectly inject instances into the ServerEndpoint like this (line 12 and 13).

<pre class="brush: java; title: ; notranslate" title="">Server server = new Server();
ServerConnector connector = new ServerConnector(server);
connector.setPort(1234);
server.addConnector(connector);

ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
context.setContextPath("/");
server.setHandler(context);

ServerContainer wscontainer;
wscontainer = WebSocketServerContainerInitializer.configureContext(context);
wscontainer.addEndpoint(new ServerEndpointBaseConfigInstanceFactory&lt;ServerEndpoint&gt;(ServerEndpoint.class, 
	(t) -&gt; new ServerEndpoint((condition) ? new SomeDependentComponentA() : new SomeDependentComponentB())));
</pre>

[0]: I checked with annotated ServerEndpoints, only. I am not sure if this still works that way if your ServerEndpoint extends Endpoint.