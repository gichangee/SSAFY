# API 문서 만들기

API 문서 생성

GET /posts/{postId} → 단건 조회

POST /posts → 게시글 등록

클라이언트 입장에서는 어떤 API 있는지 모름

Spring REST Docs

운영코드에 영향 X

코드 수정를 수정했을 때 문서를 수정 안하게 되면 코드와 문서  서로 스펙이 일치하지않음

문서에 대한 신뢰성이 갈수록 떨어짐

하지만 Rest Docs는 Test 케이스 실행 → 문서를 자동으로 생성

즉 테스트코드가 통과가 안되면 문서가 생성이 안됨

문서화가 안된부분이 있으면 알려줌

build → bootJar 실행

# 기본설정

build.gradle

```jsx
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.2'
    id 'io.spring.dependency-management' version '1.1.6'
    id "org.asciidoctor.jvm.convert" version "3.3.2"
}

group = 'com.gildong.api'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    asciidoctorExt
}

repositories {
    mavenCentral()
}

ext {
    asciidocVersion = "3.0.0"
    snippetsDir = file('build/generated-snippets')
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    implementation 'com.querydsl:querydsl-core'
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'

    asciidoctorExt "org.springframework.restdocs:spring-restdocs-asciidoctor:${asciidocVersion}"
    testImplementation "org.springframework.restdocs:spring-restdocs-mockmvc:${asciidocVersion}"

    annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}

test {
    outputs.dir snippetsDir
}

asciidoctor {
    inputs.dir snippetsDir
    configurations 'asciidoctorExt'
    dependsOn test

}

bootJar {
    dependsOn asciidoctor

    copy {
        from asciidoctor.outputDir
        into "src/main/resources/static/docs"
    }
}

```

src/docs/asciidoc/index.adoc

```jsx
= 기돌로그 API
:toc:

== 글 단건 조회

=== 요청
include::{snippets}/index/http-request.adoc[]

=== 응답
include::{snippets}/index/http-response.adoc[]

=== CURL
include::{snippets}/index/curl-request.adoc[]

```

테스트

```jsx

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@ExtendWith(RestDocumentationExtension.class)
public class PostControllerDocTest {
    private MockMvc mockMvc;

    @Autowired
    private PostRepository postRepository;

    @BeforeEach
    void setUp(WebApplicationContext webApplicationContext, RestDocumentationContextProvider restDocumentation) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .apply(documentationConfiguration(restDocumentation))
                .addFilters(new CharacterEncodingFilter("UTF-8", true))
                .build();
    }

    @Test
    @DisplayName("글 단건 조회 테스트")
    void test1() throws Exception {

        //given
        Post post = Post.builder()
                .title("제목")
                .content("내용")
                .build();

        postRepository.save(post);

        this.mockMvc.perform(get("/posts/{postId}",1L)
                        .accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andDo(document("index"));
    }
}

```

# 요청, 응답필드 및 커스터마이징

테스트

```jsx

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureRestDocs(uriScheme = "https", uriHost = "api.hodolman.com",uriPort = 443)
@ExtendWith(RestDocumentationExtension.class)
public class PostControllerDocTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("글 단건 조회 테스트")
    void test1() throws Exception {

        //given
        Post post = Post.builder()
                .title("제목")
                .content("내용")
                .build();

        postRepository.save(post);

        this.mockMvc.perform(RestDocumentationRequestBuilders.get("/posts/{postId}",1L)
                        .accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andDo(document("post-inquiry", RequestDocumentation.pathParameters(
                            RequestDocumentation.parameterWithName("postId").description("게시글 ID")
                        ),
                        PayloadDocumentation.responseFields(
                                PayloadDocumentation.fieldWithPath("id").description("게시글 ID"),
                                PayloadDocumentation.fieldWithPath("title").description("제목"),
                                PayloadDocumentation.fieldWithPath("content").description("내용")
                        )
                ));
    }

    @Test
    @DisplayName("글 등록 테스트")
    void test2() throws Exception {

        //given

        PostCreate request = PostCreate.builder()
                .title("호돌맨")
                .content("반포자이.")
                .build();

        String json = objectMapper.writeValueAsString(request);

        //expected
        this.mockMvc.perform(RestDocumentationRequestBuilders.post("/posts")
                        .contentType(MediaType.APPLICATION_JSON)
                        .accept(MediaType.APPLICATION_JSON)
                        .content(json))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(status().isOk())
                .andDo(document("post-create",
                        PayloadDocumentation.requestFields(
                                PayloadDocumentation.fieldWithPath("title").description("제목")
                                        .attributes(Attributes.key("constraint").value("좋은 제목 입력해주세용")),
                                PayloadDocumentation.fieldWithPath("content").description("내용").optional()
                        )
                ));
    }
}

```

src/test/resources/org/springframework/restdocs/templates/asciidoctor/request-fields.snippet

```jsx
|===
    |Path|Type|Description|Optional|Constraint

    {{#fields}}
            |{{path}}
            |{{type}}
            |{{description}}
            |{{#optional}}Y{{/optional}}
            |{{#constraint}} {{.}} {{/constraint}}
    {{/fields}}
|===
```

/src/docs/asciidoc/index.adoc

```jsx
= 기돌로그 API
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

== 글 단건 조회

=== 요청
include::{snippets}/post-inquiry/http-request.adoc[]

include::{snippets}/post-inquiry/path-parameters.adoc[]

=== 응답
include::{snippets}/post-inquiry/http-response.adoc[]

include::{snippets}/post-inquiry/response-fields.adoc[]

== 글 작성

=== 요청
include::{snippets}/post-create/http-request.adoc[]

include::{snippets}/post-create/request-fields.adoc[]

=== 응답
include::{snippets}/post-create/http-response.adoc[]

```