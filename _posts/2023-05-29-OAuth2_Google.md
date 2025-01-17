---
layout: single
title: "Spring Security, OAuth2.0으로 구글 로그인"
toc: true
toc_sticky: true
categories: 
    - Spring
tags: 
    - Spring
    - Boot
    - Security
    - OAuth2.0
    - Social Login
    - Google
---

# 프로젝트 전체 구조    
시작하기 앞서 최종 산출물의 구조를 보여드리겠습니다.   

![structure](/images/2023-05-29-OAuth2_Google/2023-05-29-02-48-07.png) 

<br>
<br>

# Gradle   
## Spring boot 버전   

>`3.0.7`

## dependencies   

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'

    testImplementation 'org.springframework.security:spring-security-test'
}
```   

구글 로그일을 하기 위해서 security, oauth2-client, thymeleaf-extras-springsecurity6을 추가해줍니다.   
기본적인 jpa, thymeleaf, web, lombok, mysql은 적어 놓진 않았습니다.

<br>
<br>

# Google Oauth 2.0    
## Client ID 발급

![GCP_step1](/images/2023-05-29-OAuth2_Google/2023-05-29-03-05-43.png)   

[구글 클라우드(GCP)](https://console.cloud.google.com/?hl=ko)로 이동해서 빨간 박스 부분을 눌러 줍니다.

<br>
<br>

![GCP_step2](/images/2023-05-29-OAuth2_Google/2023-05-29-03-12-57.png)


누르면 아래와 같은 화면이 나올텐데요. 오른쪽 위 `새 프로젝트`를 클릭해줍니다.

<br>
<br>

![GCP_step3](/images/2023-05-29-OAuth2_Google/2023-05-29-03-15-03.png)   

프로젝트 이름을 작성한 뒤 만들기를 눌러주세요.

<br>
<br>

![GCP_step4](/images/2023-05-29-OAuth2_Google/2023-05-29-03-17-31.png)   

만드셨다면 왼쪽 상단에 프로젝트 이름이 방금 생성한 것과 동일한지 확인 후 `API 및 서비스 > OAuth 동의 화면`을 눌러주세요.

<br>
<br>

![GCP_step5](/images/2023-05-29-OAuth2_Google/2023-05-29-03-20-33.png)

`외부`로 해주신 뒤 만들기를 눌러주세요.

<br>
<br>

![GCP_step6](/images/2023-05-29-OAuth2_Google/2023-05-29-03-25-38.png)

`앱 이름`: 나중에 구글 로그인할 때 화면에 띄어주는 이름입니다.   
`사용자 인증 이메일`: 해당 프로젝트의 문의 메일입니다. 만약 실제로 운영한다면 대표 이메일을 작성하시면 됩니다.   
`앱 로고`: 로그인할 때 띄어주는 프로젝트 로고입니다. (앱 게시를 하기 전까지는 보이지 않습니다!)

<br>
<br>

![GCP_step7](/images/2023-05-29-OAuth2_Google/2023-05-29-03-32-23.png)

현재는 테스트이기 때문에 도메인은 비워두고 `개발자 이메일`만 적어주시고 저장후 계속해주세요.

<br>
<br>

![GCP_step7](/images/2023-05-29-OAuth2_Google/2023-05-29-03-35-17.png)

다음은 범위입니다. 저희 프로젝트가 로그인하는 사용자의 정보를 어느정도의 범위로 가져롱 것인지를 정하는 단계입니다.   
범위 추가 또는 삭제를 눌러주세요.

<br>
<br>

![GCP_step8](/images/2023-05-29-OAuth2_Google/2023-05-29-03-36-20.png)

간단하게 사용자의 email과 profile만 가져오겠습니다.   
저장 후 계속해주세요.

<br>
<br>

`테스트 사용자` 단계는 그대로 계속 넘기시면됩니다.

<br>
<br>

![GCP_step9](/images/2023-05-29-OAuth2_Google/2023-05-29-03-40-28.png)

마지막으로 요약보고 맞는지 확인 후 왼쪽 `사용자 인증 정보`를 눌러줍니다.

<br>
<br>

![GCP_step10](/images/2023-05-29-OAuth2_Google/2023-05-29-03-43-10.png)

`사용자 인증 정보 만들기 > OAuth 클라이언트 ID`를 눌러줍니다.

<br>
<br>

![GCP_step11](/images/2023-05-29-OAuth2_Google/2023-05-29-03-46-38.png)

`애플리케이션`: 웹 애플리케이션   
`이름`: 본인이 식별할 이름   
`승인된 리디렉션 URI`: "http://localhost:8080/login/oauth2/code/google"   

"http://localhost:8080/login/oauth2/code/google" 주소는 Spring에서 자동으로 생성되는 주소입니다. 다음으로 만들기를 해주세요.

<br>
<br>

![GCP_step12](/images/2023-05-29-OAuth2_Google/2023-05-29-03-51-12.png)

생성한 클라이언트 ID와 클라이언트 보안 비밀번호를 

<br>
<br>

## application 설정

> application-oauth.yaml
```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: 키입력
            client-secret: 키입력
            scope:
              - profile
              - email
```

`application.yaml` 파일과 같은 폴더에 `application-oauth.yaml`를 위와 같이 생성해줍니다. `application-oauth.yaml`에 `client-id`와 `client-secret`에 방급 발급 받은 값을 넣어줍니다.

<br>
<br>

> application.yaml
>```yaml
>spring:
>  profiles:
>    include: oauth
>```

`application.yaml`에서 `application-oauth.yaml`을 불러오는 방식으로 작성 해줍니다.   
github에 올라갈 땐 `application-oauth.yaml`을 .gitignore에 등록해줍니다.

<br>
<br>

# 도메인
## Member(사용자 정보)

> Member Class
>```java
>@Entity
>@Getter @Setter
>@NoArgsConstructor
>public class Member {
>
>    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
>    @Column(name = "member_id")
>    private Long id; //기본키
>    private String name; //유저 이름
>    private String email; //유저 구글 이메일
>    private String provider; //공급자 (google, facebook ...)
>    private String providerId; //공급 아이디
>
>    @Enumerated(EnumType.STRING)
>    @Column(nullable = false)
>    private Role role; //유저 권한 (관리자, 고객)
>
>    @Builder
>    public Member(String name, String email, Role role, String provider, String providerId) {
>        this.name = name;
>        this.email = email;
>        this.role = role;
>        this.provider = provider;
>        this.providerId = providerId;
>    }
>}
>```

<br>
<br>

## Role(사용자 권한)

> Role Enum
>```java
>@Getter
>@RequiredArgsConstructor
>public enum Role {
>
>    ADMIN("ROLE_ADMIN", "관리자"),
>    USER("ROLE_USER", "고객");
>
>    private final String key;
>    private final String title;
>}
>```

<br>
<br>

# Repository
## MemberRepository

> MemberRepository Interface
>```java
>public interface MemberRepository extends JpaRepository<Member, Long> {
>
>    public Member getOne(Long id);
>    public List<Member> findByEmail(String email);
>}
>```

`MemberRepository`는 PK로 검색과 Email로 검색만 구현해주었습니다.

<br>
<br>

# DTO
## PrincipalDetails (OAuth2User의 구현 클래스)

> PrincipalDetails
>```java
>@Getter
>@Setter
>public class PrincipalDetails implements OAuth2User {
>    private Member member;
>    private Map<String, Object> attributes;
>
>    // OAuth 로그인
>    public PrincipalDetails(Member member, Map<String, Object> attributes) {
>        this.member=member;
>        this.attributes=attributes;
>    }
>
>
>    // 해당 Member의 권한을 리턴하는 곳
>    @Override
>    public Collection<? extends GrantedAuthority> getAuthorities() {
>        Collection<GrantedAuthority> authorities = new ArrayList<>();
>
>        authorities.add(new SimpleGrantedAuthority(member.getRole().getKey()));
>
>        return authorities;
>    }
>
>    @Override
>    public String getName() {
>        return getAttribute("name");
>    }
>}
>```

`PrincipalDetails`은 Member 클래스의 권한 부분을 Member 클래스에 맞게 `getAuthorities()`을 Override해줍니다. 또한 `PrincipalDetails`은 다양하게 활용될 수 있고 Member 객체를 넣지 않고 name, email, role 등 자유롭게 넣고 사용할 수 있습니다.

<br>
<br>

# Config
## SecurityConfig

> SecurityConfig Class
> ```java
> @Configuration
> @RequiredArgsConstructor
> @EnableWebSecurity // 스프링 시큐리티 필터가 스프링 필터체인에 등록
> public class SecurityConfig {
>     private final OAuth2MemberService oAuth2MemberService;
> 
>     @Bean
>     public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception{
>         return httpSecurity
>                 .httpBasic().disable() // httpBasic 방식 대신 Jwt를 사용하기 때문에 disable로 설정
>                 .csrf().disable() // 세션 방식 로그인이 아니기 때문에 불필요한 위조방지를 꺼야한다.
>                 .cors() // 다른 도메인 (도메인 간 요청)을 가진 리소스에 액세스 할 수 있게하는 보안 메커니즘
>             .and()
>                 .authorizeRequests()
>                 .requestMatchers("/private/**").authenticated() //private로 시작하는 uri는 로그인 필수
>                 .requestMatchers("/admin/**").hasRole(Role.ADMIN.name()) //admin으로 시작하는 uri는 관리자 계정만 접근 가능
>                 .anyRequest().permitAll() //나머지 uri는 모든 접근 허용
>             .and()
>                 .logout()
>                 .logoutSuccessUrl("/") // logout 성공시 이동할 url 설정
>             .and()
>                 .oauth2Login()
>                 .loginPage("/loginForm") //로그인이 필요한데 로그인을 하지 않았다면 이동할 uri 설정
>                 .defaultSuccessUrl("/") //OAuth 구글 로그인이 성공하면 이동할 uri 설정
>                 .userInfoEndpoint()//로그인 완료 후 회원 정보 받기
>                 .userService(oAuth2MemberService)
>             .and().and().build(); //로그인 후 받아온 유저 정보 처리
>     }
> }
> ```

`SecurityConfig`는 사용자가 웹에 접속할 때 작동하는 필터이며, 마지막에 `oAuth2MemberService`로 로그인 후 사용자 정보를 넘겨줍니다.   
라인마다 주석으로 설명드리는 것이 더 이해하기 빠를 것 같아서 주석으로 자세히 작성했습니다. 읽어보시면 쉽게 이해 가능할 겁니다.

<br>
<br>

# Service
## OAuth2MemberService

> OAuth2MemberService Class   
> ```java
> @Service 
> @RequiredArgsConstructor
> @Transactional(readOnly = true)
> public class OAuth2MemberService extends DefaultOAuth2UserService {
>     private final MemberRepository memberRepository;
>     @Override
>     @Transactional
>     public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
>         OAuth2User oAuth2User = super.loadUser(userRequest);
> 
>         // name 생성
>         String provider = userRequest.getClientRegistration().getRegistrationId(); //google
>         String providerId = oAuth2User.getAttribute("sub");
>         String name = oAuth2User.getAttribute("name");
>         String email = oAuth2User.getAttribute("email");
>         Role role = Role.USER; //일반 유저
> 
>         List<Member> members = memberRepository.findByEmail(email);
>         Member member;
>         if (members.isEmpty()) { //최초 로그인
>             member = Member.builder()
>                     .name(name)
>                     .email(email)
>                     .role(role)
>                     .provider(provider)
>                     .providerId(providerId).build();
>             memberRepository.save(member);
>         } else{ // 기존 고객
>             member = members.get(0);
>         }
> 
>         return new PrincipalDetails(member,  oAuth2User.getAttributes());
>     }
> }
> ```

`OAuth2MemberService`은 로그인한 정보를 받아서 DB에 저장하거나 기존에 있는 사용자라면 DB에서 불러온 뒤 `PrincipalDetails`을 반환해 줍니다.

<br>
<br>

## MemberService

> MemberService Class
> ``` java 
> @Service
> @Transactional(readOnly = true)
> @RequiredArgsConstructor //final이 있는 필드만 가지고 생성자를 자동 생성
> public class MemberService {
> 
>     private final MemberRepository memberRepository;
> 
>     @Transactional
>     public Member updateMember(Long memberId) {
>         Member member = memberRepository.getOne(memberId);
>         member.setName("NEW_Name");
>         member.setRole(Role.ADMIN); // 권한은 업데이트가 바로 적용이 안됨 다시 로그인 필요
>         return member;
>     }
> }
> ```

`MemberService`의 `updateMember()`는 사용자가 로그인 후 정보를 수정 했을 때 DB를 수정하는 테스트 로직입니다.

<br>
<br>

# Controller
## HomeController

> HomeController Class
> ``` java
> @Controller
> @Slf4j
> public class HomeController {
> 
>     @RequestMapping("/")
>     public String home(Model model){
>         log.info("home controller");
>         return "home";
>     }
> }
> ```

## OAuthController

> OAuthController Class
> ``` java
> @Controller
> @RequiredArgsConstructor
> public class OAuthController {
> 
>     private final MemberService memberService;
>     @GetMapping("/loginForm")
>     public String home() {
>         return "loginForm";
>     }
> 
>     @GetMapping("/private")
>     public String privatePage() {
>         return "privatePage";
>     }
>     @GetMapping("/admin")
>     public String adminPage() {
>         return "adminPage";
>     }
> 
>     @GetMapping("/update")
>     public String update(@AuthenticationPrincipal PrincipalDetails principal) {
>         if (principal != null){
>             Long id = principal.getMember().getId();
>             Member member = memberService.updateMember(id);
>             principal.setMember(member); // 권한은 업데이트가 바로 적용이 안됨 다시 로그인 필요
>         }
>         return "redirect:/";
>     }
> }
> ```

`OAuthController`은 로그인 페이지와 권한 설정이 되어있는 페이지와 사용자 정보를 업데이트하는 기능으로 구성되어 있습니다.

<br>
<br>

# HTML (Thymeleaf)

HTML은 매우 간단하게 작성했습니다. 여기서 중요한 것은 `xmlns:sec` 네임 스페이스를 등록한 뒤 `sec:authorize`, `sec:authentication` 부분을 이용해서 사용자가 로그인 상태인지와 로그인한 정보를 가져와서 출력해줄 수 있습니다.   
`<a sec:authentication="principal.member.name"/>` 이 부분에서 principal은 DTO에서 생성한 `PrincipalDetails` 클래스입니다. `PrincipalDetails`를 수정해서 원하는 로그인 정보를 바로 사용할 수 있습니다.

## home

> home HTML
> ``` html
> <!DOCTYPE html>
> <html xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
> <head>
>     <meta charset="UTF-8">
>     <title>Title</title>
> </head>
> <body>
> <ul class="nav ">
>     <li sec:authorize="isAnonymous()">
>         <a href="/loginForm">로그인</a>
>     </li>
>     <li sec:authorize="isAuthenticated()">
>           <span>
>           <a sec:authentication="principal.member.name"/>
>           (<a sec:authentication="principal.member.email"/>)님 안녕하세요!
>           </span>
>         <a href="/logout">로그아웃</a>
>     </li>
>     <li sec:authorize="isAuthenticated()">
>         <a href="/private">로그인한 사람만 접속 가능 사이트</a>
>     </li>
>     <li sec:authorize="isAuthenticated()">
>         <a href="/admin">어드민만 접속 가능한 사이트</a>
>     </li>
>     <li sec:authorize="isAuthenticated()">
>         <a href="/update">이름, 권한 업데이트</a>
>     </li>
> </ul>
> </body>
> </html>
> ```

<br>
<br>

## loginForm

> loginForm HTML
> ``` html
> <!DOCTYPE html>
> <html lang="en">
> 
> <head>
>     <meta charset="UTF-8">
>     <title>Social Login</title>
> 
> </head>
> 
> <body>
> 
> <h1>Social Login</h1>
> 
>     <a class="button button--social-login button--googleplus" href="/oauth2/authorization/google">Login With Google +</a>
> 
> </body>
> 
> </html>
> ```

<br>
<br>

## privatePage

> privatePage HTML
> ``` html
> <!DOCTYPE html>
> <html lang="en">
> <head>
>     <meta charset="UTF-8">
>     <title>Title</title>
> </head>
> <body>
> private
> </body>
> </html>
> ```

<br>
<br>

## adminPage

> adminPage HTML
> ``` html
> <!DOCTYPE html>
> <html lang="en">
> <head>
>     <meta charset="UTF-8">
>     <title>Title</title>
> </head>
> <body>
> admin
> </body>
> </html>
> ```

<br>
<br>

# 결과

> 로그인 전
> 
> ![로그인 전](/images/2023-05-29-OAuth2_Google/2023-05-29-05-15-52.png)


> 로그인 후
> 
> ![로그인 후](/images/2023-05-29-OAuth2_Google/2023-05-29-05-19-04.png)

> DB
> 
> ![DB](/images/2023-05-29-OAuth2_Google/2023-05-30-06-52-57.png)

<br>
<br>
   
Github   

<https://github.com/HYLogs/OAuth2.0/tree/GoogleOAuth>

<br>
<br>

참고   
스프링부트 시큐리티 & JWT 강의   
 
<https://inf.run/Legg>