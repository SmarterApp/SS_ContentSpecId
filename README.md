# Content Spec ID Conversion Library

Java Library to handle conversion of content spec ids between enhanced and legacy formats.  The .NET version of the application can be found [here](https://github.com/SmarterApp/SC_ContentSpecId)

* [Legacy Format Documentation](https://www.smarterapp.org/documents/InterpretingSmarterBalancedStandardIDs.html)
* [Enhanced Format Documentation](https://www.smarterapp.org/documents/ContentSpecificationIdFormats.pdf)

## Technologies

* Java 8
* Project Lombok 
* Gradle
* JGitflow

## Development Setup
This section lists out the necessary steps if you want to contribute to this project including how to build and run the project.

### Gradle Build
Installing Gradle separately is not required. Instead use the Gradle wrapper.  

Developers are suggested to use the alias ```gw``` as it is more convenient than typing  ```gradlew```.

Build: ```./gw clean build```

Build skip tests: ```./gw clean build -x test```

### Enable Annotation Processing in IntelliJ IDEA
1. IntelliJ IDEA -> Preferences
2. Build, Execution, Deployment -> Compiler -> Annotation Processors
3. Check "Enable annotation processing"
4. OK

### Publishing

__Local Publishing__

When developing it is only necessary to publish to your local Maven repository.  
Dependent apps will find it locally assuming they are configured to look there.

```./gw publishToMavenLocal``` or the shorthand ```./gw pubTML```

__Artifactory Publishing__

Publishing the snapshot or release to Artifactory is possible.

You must set in your ~/.gradle/gradle.properties

* artifactoryUser=
* artifactoryPassword=

The project defines in its gradle.properties these optional properties which you can override in 
 ~/.gradle/gradle.properties when necessary.

* artifactoryUrl=
* artifactorySnapshotPublish=
* artifactoryReleasePublish=

The artifact is published by default to libs-snapshots-local if the version is set with -SNAPSHOT.
It publishes to libs-releases-local if the version is not a snapshot.  The version property is defined
in gradle.properties.

__Publish Command__

```./gw artifactoryPublish```

## API Documentation and Examples
This section gives examples of how to use the public APIs within this library.  This is only intended to have basic examples.  More detailed examples can be found looking through the source control and test suite.

### Convert a Content Spec ID string to another format.
The primary purpose of the Content Spec ID library is to convert IDs from one format to another.
The first step is to create a ContentSpecIdConverter and use it to parse an existing ID.

```java
String inputIdString = "SBAC-ELA-v1:1-LT|7-4|4.L.5";
ContentSpecIdConverter converter = new ContentSpecIdConverter();
ContentSpecId id = converter.parse(inputIdString);
``` 

This will throw a ValidationException if the input ID string cannot be parsed as a Content Spec ID.

A second parse method allows passing the default grade level in addition to the ID string. This supports parsing
legacy IDs, which sometimes omit grade information. 

```java
String inputIdString = "SBAC-ELA-v1:1-LT|7|4.L.5";
ContentSpecIdConverter converter = new ContentSpecIdConverter();
ContentSpecId id = converter.parse(inputIdString, ContentSpecGrade.G4);
``` 

The second and final step in the conversion process is to format the ContentSpecId object into a String with
the desired output format. The converter has several format methods to accomplish this task:

```java
String outputIdString;

// Formats the id into enhanced format
outputIdString = converter.format(id);

// Formats the id into default legacy format based on the subject.
outputIdString = converter.formatLegacy(id); 

// Formats the id into the given format. Here that is Enhanced.
outputIdString = converter.format(id, ContentSpecFormat.ENHANCED); 
```

Any of the format methods can throw ValidationExceptions if the ID cannot be converted into the requested format.
This usually means that the id object is missing information needed by the requested output format.

The default legacy format depends on the subject. For ELA, it is ELA, v1 and for Math, it is Math, v4. 
If a specific output format is specified, it must be one that is compatible with the ID's subject. If not,
this will also cause a ValidationExcpetion to be thrown.

### Determining the format of an ID string
In addition to the parse and format methods, the ContentSpecIdConverter has a method to determine 
the format of a given ID string:
```java
String inputIdString = "SBAC-ELA-v1:1-LT|7|4.L.5";
ContentSpecIdConverter converter = new ContentSpecIdConverter();
ContentSpecFormat format = converter.getFormatType(inputIdString);
// format is ContentSpecFormat.ELA_V1;
```
The getFormatType() method will throw a ValidationFormat if the ID string is not in a recognizable format.

### Building an ID from component parts
It is also possible to build an ID object from its component parts using the ContentSpecIdBuilder.
```java
// The builder is initialized with the bare minimum information to create a valid ID.
ContentSpecIdBuilder builder = ContentSpecIdBuilder.getBuilder(
        ContentSpecSubject.MATH, ContentSpecGrade.G4);

// Additional parts are added and then the build method procduces a ContentSpecId object
ContentSpecId id = builder
        .claim("1")
        .target("7")
        .domain("LT")
        .ccss("4.L.5")
        .build();
```

Like the parse methods, the builder's build() method will throw a ValidationException if any of the components cannot
be converted correctly to ID fields. Once the id object is built successfully, it can be passed to the any of the
converter's format methods exactly as if it had been parsed from a String.

Note: there are some differences in the format of the component parts for Legacy and Enhanced IDs. For example,
a legacy claim string is just a number from 1 to 4, but an enhanced claim string has a C prefix, so can be C1, C2, C3 or C4.
Also, some of the ELA legacy domains have different codes from equivalent domains in the enhanced format. 
However, the builder should work correctly whether given legacy or enhanced component strings.

### Converting Strings to enum types
For convenience, several of the enum types have fromString() methods to get a type based on input string. These input
strings can be either in legacy or enhanced format.

For example, a grade type is needed to create a builder, and this can easily be built
from a string:

```java
ContentSpecGrade grade;

// Grade is type ContentSpecGrade.G4
grade = ContentSpecGrade.fromString("G4");

// Grade from legacy string is also ContentSpecGrade.G4
grade = ContentSpecGrade.fromString("4");

// High school specified in enhanced format is ContentSpecGrade.GHS
grade = ContentSpecGrade.fromString("GHS");

// High school specified in legacy format is also ContentSpecGrade.GHS
grade = ContentSpecGrade.fromString("11");
```

ContentSpecClaim and DomainCode types can also be built from strings, but these are more useful internally to the 
library and may not have any practical use to outside callers. 

## Command Line API
This is a placeholder section for a planned enhancement to the library
to provide a command line API to expedite QA and acceptance testing.