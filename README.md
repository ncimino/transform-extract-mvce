# MVCE to test extraction transforms for C++ projects

This example came into existence while trying to have a CMake project depend on a cpp-library generated dependency. 
While connecting up a transform that would auto-extract the cpp-library generated header archive for the CMake plugin
the following error was encountered:

      > No matching variant of com.test:utilities:1.2.3 was found. The consumer was configured to find attribute 'org.gradle.usage' with value 'cplusplus-api', attribute 'artifactType' with value 'directory' but:
          - Variant 'api' capability com.test:utilities:1.2.3 declares attribute 'org.gradle.usage' with value 'cplusplus-api':
              - Incompatible because this component declares attribute 'artifactType' with value 'zip' and the consumer needed attribute 'artifactType' with value 'directory'

The error doesn't make sense because a transform is registered almost identical to how the cpp-library defines it's
transform.

Documentation on artifact transform selection and execution outlines how to do this:

* https://docs.gradle.org/current/userguide/artifact_transforms.html#artifact_transform_selection_and_execution

Instead of using an actual CMake plugin as the consumer here, we're just using a very simple setup that reproduces. 
The CMake plugin code is very similar to what exists here:

* https://github.com/gradle/native-samples/tree/master/cpp/cmake-library/build-wrapper-plugin/src/main/java/org/gradle/samples/plugins

# To test

Publish the dependency just to see it:

    ./gradlew :lib:publishAllPublicationsToExampleRepository

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
