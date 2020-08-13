# intro
- **[주의] 삽질의 기록, 실질적인 해결은 하지 못함**
- android 에서 google cloud speech API 를 사용중이다. 
- android sample code 를 보면, speech API .proto 파일을 직접 가져다가 protoc 해서 사용하고 있다.
- java 에서는 gradle dependency 에 등록하고 사용하면 된다고 하는데, 굳이 직접 compile 까지 해가면서 사용할 이유가 있을까?
- google cloud java sample 참고해서 작업해보려는데 아래와 같은 의미심장한 말이 적혀있음, 지금도 유효한 말인지 알 길은 없음.(하지만 유효해 보인다..)
> [`참고: Cloud 자바 클라이언트 라이브러리는 현재 Android를 지원하지 않습니다.`](https://cloud.google.com/speech-to-text/docs/quickstart-client-libraries)

# progress
## expected problem
- google speech API 가 어느 순간 backward compatibility 를 서서히 잃기 시작한다면?
  - e.g. : required parameter `a` 를 추가했는데, 전달할 방법이 없다.
- 노후화된 protoc 및 각종 dependency(grpc-core, javalite, java..) 를 사용하고있는데 build.gradle 은 syntax 도 바뀌었고, 최신화된 문법으로 맞춰서 정리해야 될 것 같은데.. 
```
Android projects: no default output will be added. Since Protobuf 3.0.0, the lite runtime is the recommended Protobuf library for Android.
For Protobuf versions from 3.0.x through 3.7.x, lite code generation is provided as a protoc plugin (protobuf-lite). Example:
---
dependencies {
  // You need to depend on the lite runtime library, not protobuf-java
  compile 'com.google.protobuf:protobuf-lite:3.0.0'
}

protobuf {
  protoc {
    // You still need protoc like in the non-Android case
    artifact = 'com.google.protobuf:protoc:3.7.0'
  }
  plugins {
    javalite {
      // The codegen for lite comes as a separate artifact
      artifact = 'com.google.protobuf:protoc-gen-javalite:3.0.0'
    }
  }
  generateProtoTasks {
    all().each { task ->
      task.builtins {
        // In most cases you don't need the full Java output
        // if you use the lite output.
        remove java
      }
      task.plugins {
        javalite { }
      }
    }
  }
}
---
Starting from Protobuf 3.8.0, lite code generation is built into protoc's "java" output. Example:
---
dependencies {
  // You need to depend on the lite runtime library, not protobuf-java
  compile 'com.google.protobuf:protobuf-javalite:3.8.0'
}

protobuf {
  protoc {
    artifact = 'com.google.protobuf:protoc:3.8.0'
  }
  generateProtoTasks {
    all().each { task ->
      task.builtins {
        java {
          option "lite"
        }
      }
    }
  }
}
---
```
  - reference: https://github.com/google/protobuf-gradle-plugin

  
## status
- android 에서 구현한것 (proto 직접빌드)
    - channel 를 관리, channel 로부터 stub 을 직접 생성
    - 인증 직접함 (oauth.google.com?), credential 읽어서 씀

- java 의 dependency 받아서, library import 쓰는거
    - credential 을 읽어서 wrapping 된 GoogleCredentials 로 생성해서 사용
    - channel 관리 안함, 위에서 생성한 credential 을 가지고 client(stub) build 방식으로 직접 생성
        - 여기서 에러가 발생함(credential, client 생성하는 code 는 직접 생성하고 변경을 가하는것이 막혀있음)
        - 시행착오:
            - GoogleCredentialsInterceptor 를 가져다가 기존에 하던 방식처럼, channel 직접 생성해서 client 를 만들어볼까
                실패: 라이브러리에서 channel 생성 지원하지 않음.. client 를 build 해서 만들도록 되어있음.

## todo
### 확실히 모름
- java 구현해서 통신 다 한다음에, 따로 android 로 넘기면 되는거 아닌가?
    - 가장 확실한 확인방법은 다른 dependency 다 빼고 android sample 만들어서 직접 호출해보는것.
    - protoc, grpc-core, okhttp, .... , javalite, java  등이 꼬여서 그런가?
      - 이 라이브러리들 다 어차피 java/android 가릴것 없이 import 할텐데... 굳이 따지자면 javalite 랑 java 중 javalite 는 android 용이라고 함..
