---
layout: single
title: "Spring Security, OAuth2.0으로 Github 로그인"
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
    - Github
---

"[Spring Security, OAuth2.0으로 구글 로그인](https://hylogs.github.io/spring/OAuth2/)"을 보지 않으신 분은 먼저 보시고 오시면 좋습니다!

# Github Oauth2.0
## Client ID 발급

![Github_step1](/images/2023-05-30-OAuth2_Github/2023-05-31-17-02-03.png)

Github에서 오른쪽 위에 사용자를 누른 뒤 `Setting`을 눌러줍니다.   

<br>
<br>

![Github_step2](/images/2023-05-30-OAuth2_Github/2023-05-31-17-28-18.png)

`Developer settings`를 눌러주세요.

<br>
<br>

![Github_step3](/images/2023-05-30-OAuth2_Github/2023-05-31-17-29-38.png)

`OAuth Apps`를 눌러주신 후 OAuth App을 만들어 주세요.

<br>
<br>

![Github_step4](/images/2023-05-30-OAuth2_Github/2023-05-31-17-33-01.png)

`Application name`: 앱 이름   
`Hmepage URL`: http://localhost:8080   
`Authorization callback URL`: http://localhost:8080/login/oauth2/code/github   

실제 테스트이기 때문에 localhost로 해주었습니다. 추후에 변경하시면 됩니다.

<br>
<br>

## application 설정   

``` yaml
spring:
  security:
    oauth2:
      client:
        registration:
            github:
                client-id: Client ID
                client-secret: Client secrets
```

생성된 앱에서 `Client ID`와 `Client secrets`를 입력해줍니다.

<br>
<br>

# code 수정

## Member

``` java
@Entity
@Getter @Setter
@NoArgsConstructor
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id; //기본키
    private String name; //유저 이름
    private String memberProviderId; // 공급자_공급 아이디
    private String provider; //공급자 (google, github ...)
    private String providerId; //공급 아이디

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role; //유저 권한 (관리자, 고객)

    @Builder
    public Member(String name, String memberProviderId, Role role, String provider, String providerId) {
        this.name = name;
        this.memberProviderId = memberProviderId;
        this.role = role;
        this.provider = provider;
        this.providerId = providerId;
    }
}
```

기존 Member는 Email이 있었는데 Github는 Email값을 사용자가 따로 설정하지 않는다면 null값을 반환하기 때문에 Resource Server에서 공통적으로 제공하는 `provider`, `providerId`값을 이용해서 사용자 검색용 column을 생성해줍니다.

<br>
<br>

## OAuthAttributes

``` java
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String name;
    private String providerAndId;
    private String provider;
    private String providerId;
    private Role role;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String name, String providerAndId, String provider, String providerId, Role role) {
        this.attributes = attributes;
        this.name = name;
        this.providerAndId = providerAndId;
        this.provider = provider;
        this.providerId = providerId;
        this.role = role;
    }

    // Google, Github 로그인 구별
    public  static OAuthAttributes of(String provider, Map<String, Object> attributes, Role role) throws Exception {
        switch (provider) {
            case "google":
                return ofGoogle(provider, attributes, role);
            case "github":
                return ofGithub(provider, attributes, role);
            default:
                throw new Exception("지원하지 않는 로그인입니다.");
        }


    }

    // Google 로그인 처리
    private static OAuthAttributes ofGoogle(String provider, Map<String, Object> attributes, Role role) {
        return OAuthAttributes.builder()
                .attributes(attributes)
                .name((String) attributes.get("name"))
                .providerAndId(provider + "_" + attributes.get("sub"))
                .provider(provider)
                .providerId((String) attributes.get("sub"))
                .role(role)
                .build();
    }
    
    // Github 로그인 처리
    private static OAuthAttributes ofGithub(String registrationId, Map<String, Object> attributes, Role role) {
        return OAuthAttributes.builder()
                .attributes(attributes)
                .name((String) attributes.get("name"))
                .providerAndId(registrationId + "_" + attributes.get("id"))
                .provider(registrationId)
                .providerId(String.valueOf(attributes.get("id")))
                .role(role)
                .build();
    }

    public Member toEntity() {
        return Member.builder()
                .name(name)
                .memberProviderId(providerAndId)
                .role(Role.USER)
                .provider(provider)
                .providerId(providerId)
                .build();
    }
}
```

DTO로 `OAuthAttributes`를 생성해줍니다. `OAuthAttributes`는 다양한 소셜 로그인을 해도 같은 Member Class를 생성해주는 DTO입니다.

<br>
<br>

## OAuth2MemberService

 ``` java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OAuth2MemberService extends DefaultOAuth2UserService {
    private final MemberRepository memberRepository;
    @Override
    @Transactional
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(userRequest);

        System.out.println(oAuth2User.getAttributes());
        // name 생성
        String provider = userRequest.getClientRegistration().getRegistrationId(); //google
        String providerId = oAuth2User.getAttribute("sub");
        String findByMemberProviderId = provider + "_" + providerId;
        Map<String, Object> attributes = oAuth2User.getAttributes();
        Role role = Role.USER; //일반 유저

        List<Member> members = memberRepository.findByMemberProviderId(findByMemberProviderId);
        Member member;
        if (members.isEmpty()) { //최초 로그인
            try {
                member = OAuthAttributes.of(provider, attributes, role).toEntity();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
            memberRepository.save(member);
        } else{ // 기존 고객
            member = members.get(0);
        }

        return new PrincipalDetails(member,  oAuth2User.getAttributes());
    }
}
 ```

`OAuthAttributes`를 통해 Member 객체를 생성함으로 `OAuth2MemberService`도 `OAuthAttributes.of()`를 이용해서 Member 객채를 저장할 수 있도록 해줍니다.

<br>
<br>

## Home.html

``` html
<a sec:authentication="principal.member.name"/>
(<a sec:authentication="principal.member.email"/>)님 안녕하세요!
```
to 

``` html
<a sec:authentication="principal.member.name"/>님 안녕하세요!
```

member의 email이 없으므로 home.html도 위와 같이 수정해주시면 됩니다.

<br>
<br>

## loginForm.html

``` html
<a class="button button--social-login button--github" href="/oauth2/authorization/github">Login With Github</a>
```

Google 로그인 버튼 아래에 Github 로그인 버튼을 추가해줍니다.   
주소를 `/oauth2/authorization/github`로 해주시면 됩니다.

<br>
<br>

# 결과

![result](/images/2023-05-30-OAuth2_Github/2023-05-31-21-11-04.png)

구글, GitHub로 로그인해보면 다음과 같이 DB에 저장된 것을 확인할 수 있습니다.

<br>
<br>

Github

<https://github.com/HYLogs/OAuth2.0/tree/GoogleAndGithubOAuth>
