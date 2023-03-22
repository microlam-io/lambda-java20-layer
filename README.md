# AWS Lambda Java 20 Custom Runtime
![Maven Central](https://img.shields.io/maven-central/v/io.microlam/lambda-java20-layer)

Based on original work from Mark Sailes: [lambda-java17-layer](https://github.com/msailes/lambda-java17-layer)

With a few differences:

* Support for ``Java 20`` on architecture ``amd64`` and `arm64`
* Based on [AWS Corretto 20](https://github.com/corretto/corretto-20/releases) build of OpenJDK
* The maven build create 2 artifacts for each architecture
* The first part of the version is corresponding to the corretto version


The simplest way to bring the layer zip is to use the maven dependency plugin like this:

```pom.xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<version>3.2.0</version>
	<executions>
		<execution>
			<id>copy</id>
			<phase>package</phase>
			<goals>
				<goal>copy</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<artifactItems>
			<artifactItem>
				<groupId>io.microlam</groupId>
				<artifactId>lambda-java20-layer</artifactId>
				<version>20.0.0.36.1</version>
				<classifier>${java20layer.arch}</classifier>
				<type>zip</type>
			</artifactItem>
		</artifactItems>
		<outputDirectory>${project.build.directory}</outputDirectory>
		<overWriteReleases>false</overWriteReleases>
		<overWriteSnapshots>true</overWriteSnapshots>
	</configuration>
</plugin>
```

Where property ``java20layer.arch`` may be ``amd64`` or ``arm64``.

Then the simplest way to deploy it to AWS is to use the CDK:

```java
//Choose your architecture
Architecture architecture = Architecture.X86_64; // Architecture.ARM_64 or Architecture.X86_64

//Do not modify this
String arch = (architecture == Architecture.ARM_64)?"arm64":"amd64";
        
LayerVersion java20layer = new LayerVersion(this, "Java20Layer-"+ arch, LayerVersionProps.builder()
				        .layerVersionName("Java20Layer-" + arch)
				        .description("Java 20 " + arch)
				        .compatibleRuntimes(Arrays.asList(software.amazon.awscdk.services.lambda.Runtime.PROVIDED_AL2))
				        .code(Code.fromAsset("target/lambda-java20-layer-20.0.0.36.1-"+ arch + ".zip"))
				        .build());

Function handlerBuilder = Function.Builder.create(this, "SimpleLambda")			    		  
           .functionName("SimpleLambda")
           .architecture(architecture)
           .runtime(software.amazon.awscdk.services.lambda.Runtime.PROVIDED_AL2)
           .layers(Collections.singletonList(java19layer))
           .handler("example.lambda.SimpleLambda")
           .memorySize(512)
           .timeout(Duration.seconds(20))
           .code(Code.fromAsset("target/simple-lambda-1.0-SNAPSHOT-aws-lambda.zip")).build();
```





