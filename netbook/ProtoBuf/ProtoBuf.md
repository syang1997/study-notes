| .proto Type |                            Notes                             | C++ Type | Java Type  | Python Type[2] | Go Type  |
| :---------: | :----------------------------------------------------------: | :------: | :--------: | :------------: | :------: |
|   double    |                                                              |  double  |   double   |     float      | *float64 |
|    float    |                                                              |  float   |   float    |     float      | *float32 |
|    int32    | 使用可变长度编码。编码负数的效率低 - 如果你的字段可能有负值，请改用 sint32 |  int32   |    int     |      int       |  *int32  |
|    int64    | 使用可变长度编码。编码负数的效率低 - 如果你的字段可能有负值，请改用 sint64 |  int64   |    long    |  int/long[3]   |  *int64  |
|   uint32    |                       使用可变长度编码                       |  uint32  |   int[1]   |  int/long[3]   | *uint32  |
|   uint64    |                       使用可变长度编码                       |  uint64  |  long[1]   |  int/long[3]   | *uint64  |
|   sint32    | 使用可变长度编码。有符号的 int 值。这些比常规 int32 对负数能更有效地编码 |  int32   |    int     |      int       |  *int32  |
|   sint64    | 使用可变长度编码。有符号的 int 值。这些比常规 int64 对负数能更有效地编码 |  int64   |    long    |  int/long[3]   |  *int64  |
|   fixed32   |    总是四个字节。如果值通常大于 228，则比 uint32 更有效。    |  uint32  |   int[1]   |  int/long[3]   | *uint32  |
|   fixed64   |    总是八个字节。如果值通常大于 256，则比 uint64 更有效。    |  uint64  |  long[1]   |  int/long[3]   | *uint64  |
|  sfixed32   |                         总是四个字节                         |  int32   |    int     |      int       |  *int32  |
|  sfixed64   |                         总是八个字节                         |  int64   |    long    |  int/long[3]   |  *int64  |
|    bool     |                                                              |   bool   |  boolean   |      bool      |  *bool   |
|   string    |       字符串必须始终包含 UTF-8 编码或 7 位 ASCII 文本        |  string  |   String   | str/unicode[4] | *string  |
|    bytes    |                     可以包含任意字节序列                     |  string  | ByteString |      str       |  []byte  |

```
plugins {
    id 'java'
    id 'idea'
    id "com.google.protobuf" version "0.8.15"
}
apply plugin: 'idea'
apply plugin: 'com.google.protobuf'
group 'com.demo'
version '1.0'


repositories {
    mavenCentral()
}

protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.12.0"
    }
    plugins {
        grpc {
            artifact = 'io.grpc:protoc-gen-grpc-java:1.36.0'
        }
    }
    generateProtoTasks {
        all()*.plugins {
            grpc {}
        }
    }
    generatedFilesBaseDir = "$projectDir/src/main/java"
}
clean {
    delete protobuf.generatedFilesBaseDir
}

idea {
    module {
        // proto files and generated Java files are automatically added as
        // source dirs.
        // If you have additional sources, add them here:
        sourceDirs += file("/src/main/java");
    }
}

dependencies {
    annotationProcessor 'org.projectlombok:lombok:1.18.8'
    compileOnly 'org.projectlombok:lombok:1.18.8'
    compile "com.yealink.component:xweb-spring-boot-starter:2.0.2-SNAPSHOT"
    compile "com.yealink.component:common-spring-boot-starter:2.0.3.M6-SNAPSHOT"
    compile "org.springframework.cloud:spring-cloud-starter-openfeign:2.2.3.RELEASE"
    compile "com.yealink.component:trpc-spring-boot-starter:2.0.7.M6-SNAPSHOT"
    implementation 'io.grpc:grpc-netty-shaded:1.36.0'
    implementation 'io.grpc:grpc-protobuf:1.36.0'
    implementation 'io.grpc:grpc-stub:1.36.0'
    testCompile group: 'junit', name: 'junit', version: '4.13.1'
}
repositories {
    mavenLocal()
    maven { setUrl "https://nexus.yealinkops.com/repository/maven-public/" }
    maven { setUrl "https://maven.aliyun.com/repository/gradle-plugin/" }
    maven { setUrl "http://oss.jfrog.org/artifactory/oss-snapshot-local/" }
}
```

