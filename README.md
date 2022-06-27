# Classloader problem with KubernetesClient

When KubernetesClient is initializing it uses JDK ServiceLoader to load a few various services, and they fail due to classloading problems.

This example is based on work coming into Apache Camel 3.18, and when you run the application packaged with google jib maven plugin.

It creates a docker image that runs the application as a _flat_ classpath with all the JARs listed.
The classloader in use is the JDK classloader (jdk.internal.loader.ClassLoaders$AppClassLoader@675d3402)


## Stacktrace

```
~/workspace ❯ docker run -it com.foo/acme
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
2022-06-27 05:33:31.909  INFO 1 --- [           main] org.apache.camel.main.MainSupport        : Apache Camel (Main) 3.18.0-SNAPSHOT is starting
2022-06-27 05:33:32.533  INFO 1 --- [           main] org.apache.camel.main.BaseMainSupport    : Classpath scanning enabled from base package: com.foo.acme
2022-06-27 05:33:33.143  INFO 1 --- [           main] org.apache.camel.main.BaseMainSupport    : Auto-configuration summary
2022-06-27 05:33:33.162  INFO 1 --- [           main] org.apache.camel.main.BaseMainSupport    :     [application.properties]       camel.main.sourceLocationEnabled=true
2022-06-27 05:33:33.166  INFO 1 --- [           main] org.apache.camel.main.BaseMainSupport    :     [application.properties]       camel.main.tracing=false
2022-06-27 05:33:33.166  INFO 1 --- [           main] org.apache.camel.main.BaseMainSupport    :     [application.properties]       camel.main.modeline=true
2022-06-27 05:33:33.167  INFO 1 --- [           main] org.apache.camel.main.BaseMainSupport    :     [application.properties]       camel.main.basePackageScan=com.foo.acme
2022-06-27 05:33:37.044  WARN 1 --- [           main] io.fabric8.kubernetes.client.Config      : Error reading service account token from: [/var/run/secrets/kubernetes.io/serviceaccount/token]. Ignoring.
2022-06-27 05:33:39.043  INFO 1 --- [           main] rnetes.properties.BasePropertiesFunction : KubernetesClient using masterUrl: https://kubernetes.default.svc/ with namespace: null
CTTL jdk.internal.loader.ClassLoaders$AppClassLoader@675d3402
AppCtx null
ConfigMapPropertiesFunction jdk.internal.loader.ClassLoaders$AppClassLoader@675d3402
KubernetesDeserializer jdk.internal.loader.ClassLoaders$AppClassLoader@675d3402
Exception in thread "main" java.util.ServiceConfigurationError: io.fabric8.kubernetes.api.KubernetesResourceMappingProvider: Error accessing configuration file
	at java.base/java.util.ServiceLoader.fail(Unknown Source)
	at java.base/java.util.ServiceLoader$LazyClassPathLookupIterator.parse(Unknown Source)
	at java.base/java.util.ServiceLoader$LazyClassPathLookupIterator.nextProviderClass(Unknown Source)
	at java.base/java.util.ServiceLoader$LazyClassPathLookupIterator.hasNextService(Unknown Source)
	at java.base/java.util.ServiceLoader$LazyClassPathLookupIterator.hasNext(Unknown Source)
	at java.base/java.util.ServiceLoader$2.hasNext(Unknown Source)
	at java.base/java.util.ServiceLoader$3.hasNext(Unknown Source)
	at java.base/java.util.Iterator.forEachRemaining(Unknown Source)
	at java.base/java.util.Spliterators$IteratorSpliterator.forEachRemaining(Unknown Source)
	at java.base/java.util.stream.Streams$ConcatSpliterator.forEachRemaining(Unknown Source)
	at java.base/java.util.stream.AbstractPipeline.copyInto(Unknown Source)
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(Unknown Source)
	at java.base/java.util.stream.ForEachOps$ForEachOp.evaluateSequential(Unknown Source)
	at java.base/java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateSequential(Unknown Source)
	at java.base/java.util.stream.AbstractPipeline.evaluate(Unknown Source)
	at java.base/java.util.stream.ReferencePipeline.forEach(Unknown Source)
	at io.fabric8.kubernetes.internal.KubernetesDeserializer$Mapping.registerAllProviders(KubernetesDeserializer.java:342)
	at io.fabric8.kubernetes.internal.KubernetesDeserializer$Mapping.<init>(KubernetesDeserializer.java:254)
	at io.fabric8.kubernetes.internal.KubernetesDeserializer.<clinit>(KubernetesDeserializer.java:82)
	at io.fabric8.kubernetes.client.dsl.internal.HasMetadataOperationsImpl.<init>(HasMetadataOperationsImpl.java:65)
	at io.fabric8.kubernetes.client.dsl.internal.HasMetadataOperationsImpl.<init>(HasMetadataOperationsImpl.java:38)
	at io.fabric8.kubernetes.client.ResourceHandlerImpl.operation(ResourceHandlerImpl.java:77)
	at io.fabric8.kubernetes.client.Handlers.getOperation(Handlers.java:137)
	at io.fabric8.kubernetes.client.BaseKubernetesClient.configMaps(BaseKubernetesClient.java:360)
	at org.apache.camel.component.kubernetes.properties.ConfigMapPropertiesFunction.lookup(ConfigMapPropertiesFunction.java:53)
	at org.apache.camel.component.kubernetes.properties.BasePropertiesFunction.apply(BasePropertiesFunction.java:232)
	at org.apache.camel.component.kubernetes.properties.ConfigMapPropertiesFunction.apply(ConfigMapPropertiesFunction.java:30)
	at org.apache.camel.component.properties.DefaultPropertiesParser$ParsingContext.getPropertyValue(DefaultPropertiesParser.java:247)
	at org.apache.camel.component.properties.DefaultPropertiesParser$ParsingContext.readProperty(DefaultPropertiesParser.java:177)
	at org.apache.camel.component.properties.DefaultPropertiesParser$ParsingContext.doParse(DefaultPropertiesParser.java:119)
	at org.apache.camel.component.properties.DefaultPropertiesParser$ParsingContext.parse(DefaultPropertiesParser.java:103)
	at org.apache.camel.component.properties.DefaultPropertiesParser.parseUri(DefaultPropertiesParser.java:68)
	at org.apache.camel.component.properties.PropertiesComponent.parseUri(PropertiesComponent.java:312)
	at org.apache.camel.component.properties.PropertiesComponent.parseUri(PropertiesComponent.java:168)
	at org.apache.camel.impl.engine.AbstractCamelContext.resolvePropertyPlaceholders(AbstractCamelContext.java:1881)
	at org.apache.camel.impl.engine.AbstractCamelContext.resolvePropertyPlaceholders(AbstractCamelContext.java:1874)
	at org.apache.camel.support.CamelContextHelper.parseText(CamelContextHelper.java:447)
	at org.apache.camel.reifier.AbstractReifier.parseString(AbstractReifier.java:54)
	at org.apache.camel.reifier.language.ExpressionReifier.createExpression(ExpressionReifier.java:186)
	at org.apache.camel.reifier.AbstractReifier.createExpression(AbstractReifier.java:115)
	at org.apache.camel.reifier.SetBodyReifier.createProcessor(SetBodyReifier.java:34)
	at org.apache.camel.reifier.ProcessorReifier.makeProcessor(ProcessorReifier.java:847)
	at org.apache.camel.reifier.ProcessorReifier.addRoutes(ProcessorReifier.java:588)
	at org.apache.camel.reifier.RouteReifier.doCreateRoute(RouteReifier.java:236)
	at org.apache.camel.reifier.RouteReifier.createRoute(RouteReifier.java:74)
	at org.apache.camel.impl.DefaultModelReifierFactory.createRoute(DefaultModelReifierFactory.java:49)
	at org.apache.camel.impl.DefaultCamelContext.startRouteDefinitions(DefaultCamelContext.java:862)
	at org.apache.camel.impl.DefaultCamelContext.startRouteDefinitions(DefaultCamelContext.java:750)
	at org.apache.camel.impl.engine.AbstractCamelContext.doInit(AbstractCamelContext.java:2946)
	at org.apache.camel.support.service.BaseService.init(BaseService.java:83)
	at org.apache.camel.impl.engine.AbstractCamelContext.init(AbstractCamelContext.java:2629)
	at org.apache.camel.support.service.BaseService.start(BaseService.java:111)
	at org.apache.camel.impl.engine.AbstractCamelContext.start(AbstractCamelContext.java:2648)
	at org.apache.camel.impl.DefaultCamelContext.start(DefaultCamelContext.java:262)
	at org.apache.camel.main.Main.doStart(Main.java:139)
	at org.apache.camel.support.service.BaseService.start(BaseService.java:119)
	at org.apache.camel.main.MainSupport.run(MainSupport.java:89)
	at org.apache.camel.main.MainCommandLineSupport.run(MainCommandLineSupport.java:221)
	at com.foo.acme.CamelApplication.main(CamelApplication.java:9)
Caused by: java.net.MalformedURLException: no !/ found in url spec:file:/app/libs/kubernetes-model-core-5.12.2.jar!/META-INF/services/io.fabric8.kubernetes.api.KubernetesResourceMappingProvider
	at java.base/java.net.JarURLConnection.parseSpecs(Unknown Source)
	at java.base/java.net.JarURLConnection.<init>(Unknown Source)
	at java.base/sun.net.www.protocol.jar.JarURLConnection.<init>(Unknown Source)
	at java.base/sun.net.www.protocol.jar.Handler.openConnection(Unknown Source)
	at java.base/java.net.URL.openConnection(Unknown Source)
	... 58 more
```

## Building

You can build the app with jib to a local docker (such as docker desktop)

```
mvn compile jib:dockerBuild
```

And then run via

```
docker run -it com.foo/acme
```


## jib generated files

The generated `jib-classpath-file` is as follows:

```
/app/resources:/app/classes:/app/libs/camel-main-3.18.0-SNAPSHOT.jar:/app/libs/camel-api-3.18.0-SNAPSHOT.jar:/app/libs/jakarta.xml.bind-api-2.3.3.jar:/app/libs/jakarta.activation-api-1.2.2.jar:/app/libs/camel-base-3.18.0-SNAPSHOT.jar:/app/libs/camel-core-engine-3.18.0-SNAPSHOT.jar:/app/libs/camel-base-engine-3.18.0-SNAPSHOT.jar:/app/libs/camel-management-api-3.18.0-SNAPSHOT.jar:/app/libs/camel-support-3.18.0-SNAPSHOT.jar:/app/libs/camel-util-3.18.0-SNAPSHOT.jar:/app/libs/camel-core-languages-3.18.0-SNAPSHOT.jar:/app/libs/camel-health-3.18.0-SNAPSHOT.jar:/app/libs/slf4j-api-1.7.36.jar:/app/libs/camel-dsl-modeline-3.18.0-SNAPSHOT.jar:/app/libs/camel-kamelet-3.18.0-SNAPSHOT.jar:/app/libs/camel-core-reifier-3.18.0-SNAPSHOT.jar:/app/libs/camel-core-processor-3.18.0-SNAPSHOT.jar:/app/libs/camel-core-model-3.18.0-SNAPSHOT.jar:/app/libs/camel-kubernetes-3.18.0-SNAPSHOT.jar:/app/libs/camel-cloud-3.18.0-SNAPSHOT.jar:/app/libs/camel-cluster-3.18.0-SNAPSHOT.jar:/app/libs/camel-util-json-3.18.0-SNAPSHOT.jar:/app/libs/kubernetes-client-5.12.2.jar:/app/libs/kubernetes-model-core-5.12.2.jar:/app/libs/kubernetes-model-common-5.12.2.jar:/app/libs/jackson-annotations-2.13.1.jar:/app/libs/kubernetes-model-rbac-5.12.2.jar:/app/libs/kubernetes-model-admissionregistration-5.12.2.jar:/app/libs/kubernetes-model-apps-5.12.2.jar:/app/libs/kubernetes-model-autoscaling-5.12.2.jar:/app/libs/kubernetes-model-apiextensions-5.12.2.jar:/app/libs/kubernetes-model-batch-5.12.2.jar:/app/libs/kubernetes-model-certificates-5.12.2.jar:/app/libs/kubernetes-model-coordination-5.12.2.jar:/app/libs/kubernetes-model-discovery-5.12.2.jar:/app/libs/kubernetes-model-events-5.12.2.jar:/app/libs/kubernetes-model-extensions-5.12.2.jar:/app/libs/kubernetes-model-flowcontrol-5.12.2.jar:/app/libs/kubernetes-model-networking-5.12.2.jar:/app/libs/kubernetes-model-metrics-5.12.2.jar:/app/libs/kubernetes-model-policy-5.12.2.jar:/app/libs/kubernetes-model-scheduling-5.12.2.jar:/app/libs/kubernetes-model-storageclass-5.12.2.jar:/app/libs/kubernetes-model-node-5.12.2.jar:/app/libs/okhttp-3.12.12.jar:/app/libs/okio-1.15.0.jar:/app/libs/logging-interceptor-3.12.12.jar:/app/libs/jackson-dataformat-yaml-2.13.1.jar:/app/libs/snakeyaml-1.28.jar:/app/libs/jackson-datatype-jsr310-2.13.1.jar:/app/libs/jackson-databind-2.13.1.jar:/app/libs/jackson-core-2.13.1.jar:/app/libs/zjsonpatch-0.3.0.jar:/app/libs/generex-1.0.2.jar:/app/libs/automaton-1.11-8.jar:/app/libs/openshift-client-5.12.2.jar:/app/libs/openshift-model-5.12.2.jar:/app/libs/openshift-model-clusterautoscaling-5.12.2.jar:/app/libs/openshift-model-operator-5.12.2.jar:/app/libs/openshift-model-operatorhub-5.12.2.jar:/app/libs/openshift-model-machine-5.12.2.jar:/app/libs/openshift-model-whereabouts-5.12.2.jar:/app/libs/openshift-model-monitoring-5.12.2.jar:/app/libs/openshift-model-storageversionmigrator-5.12.2.jar:/app/libs/openshift-model-tuned-5.12.2.jar:/app/libs/openshift-model-console-5.12.2.jar:/app/libs/openshift-model-machineconfig-5.12.2.jar:/app/libs/openshift-model-miscellaneous-5.12.2.jar:/app/libs/openshift-model-hive-5.12.2.jar:/app/libs/openshift-model-installer-5.12.2.jar:/app/libs/camel-rest-3.18.0-SNAPSHOT.jar:/app/libs/camel-tooling-model-3.18.0-SNAPSHOT.jar:/app/libs/camel-timer-3.18.0-SNAPSHOT.jar:/app/libs/camel-xml-io-dsl-3.18.0-SNAPSHOT.jar:/app/libs/camel-xml-io-3.18.0-SNAPSHOT.jar:/app/libs/camel-xml-io-util-3.18.0-SNAPSHOT.jar:/app/libs/camel-dsl-support-3.18.0-SNAPSHOT.jar:/app/libs/camel-kamelets-0.8.1.jar:/app/libs/jansi-2.4.0.jar:/app/libs/log4j-api-2.17.2.jar:/app/libs/log4j-slf4j-impl-2.17.2.jar:/app/libs/log4j-core-2.17.2.jar
```

On kubernetes (minikube) the application is run as
```
/usr/bin/qemu-x86_64 /opt/java/openjdk/bin/java -cp @/app/jib-classpath-file com.foo.acme.CamelApplication
```