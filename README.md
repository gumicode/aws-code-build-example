# aws-code-build-example

본 프로젝트는 aws code build 서비스를 이용하기 위한 예제 소스코드 입니다. 자세한 내용은 아래의 글을 참고 해 주세요.

[AWS Code Build](https://blog.gumicode.com/docs/aws/AWS%20CodeBuild)

## 예제 소스 코드 분석

예제 소스코드는 Java Spring Boot 프로젝트이다. 코드는 아래와 같이 매우 단순한 프로젝트 이다. 단 여기서 중요한 부분은 <code>buildspec.yml</code> 파일이다.

**HomeController.java**
```java 
@RestController
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "success home";
    }
}
```

**buildspec.yml**
```yml 
version: 0.2

env:
  variables:
    AWS_ACCESS_KEY_ID: # AWS IAM 에서 발급한 KEY ID 입력
    AWS_SECRET_ACCESS_KEY: # AWS IAM 에서 발급한 SECERT 입력
    AWS_ACCOUNT_ID: # 숫자로 이루어진 계정 ID,  https://console.aws.amazon.com/billing/home?#/account 접속후 가장 상단에 있는 계정 ID
    AWS_REGION: ap-northeast-2 # 리전 정보 , 서울리전의 경우 ap-northeast-2
    IMAGE_NAME: aws-code-build-example # 컨테이너 이미지 이름
    TAGS: latest # 컨테이너 이미지 tags

phases:
  pre_build:
    commands:
      - echo Entered the pre_build phase...
      - export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
    finally:
      - echo This always runs even if the login command fails
  build:
    commands:
      - echo Entered the build phase...
      - ./gradlew bootBuildImage --imageName=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${TAGS}
    finally:
      - echo This always runs even if the install command fails
  post_build:
    commands:
      - echo Entered the post_build phase...
      - echo Build completed on `date`
      - docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${TAGS}

cache:
  paths:
    - '/root/.m2/**/*'
```

[buildspec 빌드 사양 참조](https://docs.aws.amazon.com/ko_kr/codebuild/latest/userguide/build-spec-ref.html)

AWS CodeBuild 의 핵심은 바로 <code>buildspec.yml</code> 이다. 이 파일은 내가 입력한 명령을 CodeBuild 에서 실행 해 주는 역할을 수행한다. 위 파일은 아마존 공식 문서를 보고 직접 작성 하였다.

- **version** : buildspec 의 버전이다. 현재 최신버전은 <code>0.2</code>
- **env.variables** : 터미널 명령에 사용할 환경변수를 지정할 수 있다. 위 설명을 읽고 변수에 값을 채우도록 하자.
- **pre_build.commands** : 빌드를 실행하기 전 실행할 명령어를 입력한다. 주로 docker 에 업로드 하기 전 로그인 하기 위해 사용한다.
- **build.commands** : 실제로 docker 를 빌드하는 명령어를 입력한다. 일반적으로 Dockfile 를 정의하여 사용하지만, spring boot 2.3 버전 이후부터는 Dockfile 을 만들지 않아도 빌드하는 방법을 제공한다. gradle 명령어는 <code>bootBuildImage</code> 이다.
- **post_build.commands** : 주로 빌드가 완료된 컨테이너 이미지를 컨테이너 저장소로 업로드할 때 사용한다.

