# MVCE to test extraction transforms for C++ projects

This example became required when trying to have a CMake project depend on a
cpp-library generated dependency. While connecting up a transform 

      > No matching variant of com.test:utilities:1.2.3 was found. The consumer was configured to find attribute 'org.gradle.usage' with value 'cplusplus-api', attribute 'artifactType' with value 'directory' but:
          - Variant 'api' capability com.test:utilities:1.2.3 declares attribute 'org.gradle.usage' with value 'cplusplus-api':
              - Incompatible because this component declares attribute 'artifactType' with value 'zip' and the consumer needed attribute 'artifactType' with value 'directory'

This is while trying to register a transform almost identical to the cpp-library 
one. I'm following what is in the docs (see thread for example syntax).

Docs:

* https://docs.gradle.org/current/userguide/artifact_transforms.html#artifact_transform_selection_and_execution

This is an example of the Java code that should be registering the transform 
from ZIP_TYPE to DIRECTORY_TYPE for Usage.C_PLUS_PLUS_API:

    project.afterEvaluate(proj -> {
        proj.getConfigurations().getByName("cppCompile").getAttributes().attribute(ArtifactAttributes.ARTIFACT_FORMAT, DIRECTORY_TYPE);
    });
    project.getDependencies().registerTransform(UnzipTransform.class, variantTransform -> {
        variantTransform.getFrom().attribute(ArtifactAttributes.ARTIFACT_FORMAT, ZIP_TYPE);
        variantTransform.getFrom().attribute(Usage.USAGE_ATTRIBUTE, project.getObjects().named(Usage.class, Usage.C_PLUS_PLUS_API));
        variantTransform.getTo().attribute(ArtifactAttributes.ARTIFACT_FORMAT, DIRECTORY_TYPE);
        variantTransform.getTo().attribute(Usage.USAGE_ATTRIBUTE, project.getObjects().named(Usage.class, Usage.C_PLUS_PLUS_API));
    });

It was suggested to try:

    project.afterEvaluate(proj -> {
        proj.getConfigurations().getByName("cppCompile").getAttributes().attribute(ArtifactAttributes.ARTIFACT_FORMAT, DIRECTORY_TYPE);
    });
    project.getDependencies().registerTransform(UnzipTransform.class, variantTransform -> {
        variantTransform.getFrom().attribute(ArtifactAttributes.ARTIFACT_FORMAT, ZIP_TYPE);
        variantTransform.getTo().attribute(ArtifactAttributes.ARTIFACT_FORMAT, DIRECTORY_TYPE);
    });

Instead of using an actual CMake plugin as the consumer here, we're just using
a very simple setup that reproduces.

# To test

Cleaning your m2 repo is a good first step:

    rm -rf ~/.m2/respository/*

Publish the dependency just to see it:

    ./gradlew :lib:publishToMavenLocal

Module metadata shows:

    "variants": [
        {
            "name": "api",
            "attributes": {
                "artifactType": "zip",               // ZIP_TYPE
                "org.gradle.usage": "cplusplus-api"  // Usage.C_PLUS_PLUS_API
            },
            "files": [
                {
                    "name": "cpp-api-headers.zip",
                    "url": "lib-1.0.0-cpp-api-headers.zip",
                    "size": 367,
                    "sha512": "01fda890a845a07e563bf260345ae92f5c915246fa5bde06229866e50ee0bba146455d210e8869de68f978529a5d6174e2bed87beafb9bb435c362648c4690d7",
                    "sha256": "1f884e7c3f0551e71c989f47aaaa2ba4fcaeaa50838f25ec5cad29e38f157712",
                    "sha1": "8ba8c3035055735b9c47161daa0916658426931e",
                    "md5": "7d0379901e8f424009afe4c0d7101a66"
                }
            ]
        },

Consume the dependency using a transform:

    ./gradlew :mock-consumer:assemble

Notice the error:

    Execution failed for task ':mock-consumer:someAction'.
    > Could not resolve all files for configuration ':mock-consumer:cppCompile'.
      > Could not resolve com.company:lib:1.0.0.
        Required by:
          project :mock-consumer
        > No matching variant of com.company:lib:1.0.0 was found. The consumer was configured to find attribute 'org.gradle.usage' with value 'cplusplus-api', attribute 'artifactType' with value 'directory' but:
          - Variant 'api' capability com.company:lib:1.0.0 declares attribute 'org.gradle.usage' with value 'cplusplus-api':
            - Incompatible because this component declares attribute 'artifactType' with value 'zip' and the consumer needed attribute 'artifactType' with value 'directory'
