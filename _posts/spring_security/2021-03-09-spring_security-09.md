---
title: "Spring security - 09"
layout: single
author_profile: true
categories: 
- spring_security
---

# 🏓 JWT Spring Security를 이용한 서버 구현

## ☝ 토큰형 인증을 위한 Config 작성

### ⚾ SecurityConfig 설정

- 기존 예제는 세션을 사용한 인증이었기 때문에 기존과는 조금 다르게 SecurityConfig를 설정해야 한다.
- WebSecurityConfigurerAdapter를 상속받는 SecurityConfig를 생성하고 http를 param으로 갖는 configure를 override 해준다.

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CorsFilter corsFilter;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) 
                .and()
                .addFilter(corsFilter) // @CrossOrigin(인증이 없을 때) / 필터-> 인증이 있을 때
                .formLogin().disable() // formLogin X
                .httpBasic().disable()
                .authorizeRequests()
                .antMatchers("/api/v1/user/**") // 요청이 들어오는 경로 설정
                .access("hasRole('ROLE_USER') or hasRole('ROLE_ADMIN') or hasRole('ROLE_MANAGER')") // 여기에 명시된 조건에 해당 되어야 접근 가능
                .antMatchers("/api/v1/manager/**")
                .access("hasRole('ROLE_ADMIN') or hasRole('ROLE_MANAGER')")
                .antMatchers("/api/v1/admin/**")
                .access("hasRole('ROLE_ADMIN')")
                .anyRequest().permitAll(); // 위에 있는 것들 외에는 다 그냥 허용
    }
}
```

- ```http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)``` - 여기서는 세션이 아닌 상태가 존재하지 않는 토큰을 사용할 것이므로 **STATELESS**로 설정을 해준다.
- ```.addFilter(corsFilter)``` - api 서버로 사용하게 되면 서로 다른 도메인으로부터 요청을 받으므로 CORS 설정은 필수이다. 들어오는 모든 요청에 대해 CORS 정책을 적용시키기 위해 **Filter에 등록**을 해준다.
- ```.formLogin().disable()```- 기존 세션방식에서는 formLogin을 사용했지만 여기서는 사용하지 않으므로(form 태그 사용해서 로그인 하는 방식) disable 해준다.
- ```.antMatchers().access()```- antMatchers 내부에 해당하는 경로에 대해서 access 내부 조건을 만족해야 허용한다. 여기서는 각 조건에 대해 role을 어떤 것을 갖고 있는지 검사 후 허용한다.
- ```.anyRequest().permitAll()```- 위에 권한 설정을 한 경로 외에 다른 경로는 권한을 검사하지 않고 모두 허용한다.

### ⚽ CorsConfig 설정

- 앞서 언급한대로 api 서버는 front end 서버와 도메인이나 포트가 다를 수 있으므로 cors를 허용해주어야 한다.
- 원래 front end의 ip에 대해서만 허용해주어야 하나 여기서는 간단한 예제이므로 전부 허용하는 방식으로 구현한다.

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true); // 서버가 응답할 때 json을 js에서 처리할 수 있게 설정, flase로 하면 js로 요청해면 응답 안감
        config.addAllowedOrigin("*"); // 모든 주소(ip)로부터의 요청 허용
        config.addAllowedHeader("*"); // 모든 헤더 허용
        config.addAllowedMethod("*"); // post, get 등등 모든 메소드 허용
        source.registerCorsConfiguration("/api/**", config); // api로 시작하는 모든 경로에 대해 config 적용
        return new CorsFilter(source);
    }
}
```

- SecurityConfig에 등록할 CorsFilter를 만들어 DI를 위해 Bean으로 등록한다.
- ```config.setAllowCredentials(true)```- 자바스크립트 요청에 대해 응답을 허용하는 구문으로써 false로 설정한다면 js측의 axios나 ajax로 요청을 해도 데이터가 리턴되지 않는다.
- ```config.addAllowedOrigin("*")```- 모든 ip로부터의 요청을 허용한다.
- ```config.addAllowedHeader("*")```- 모든 요청의 모든 헤더를 허용한다.
- ```config.addAllowedMethod("*")```- post, get, put, delete 등 모든 http method를 허용한다.
- ```source.registerCorsConfiguration("/api/**", config)```- api로 시작하는 모든 엔드 포인트에 이 설정을 적용한다.
- 이후 이 위에서 살펴보았듯이 이 설정을 SecurityConfig에 등록하는데 이 방법 외에도 controller 클래스 위에 ```@CrossOrigin``` 어노테이션을 붙여 crossOrigin을 허용할 수 있다.
- 하지만 이 경우에는 인증이 필요한 경우에 적절히 인증을 할 수 없으므로 필터에 등록하는 방식을 취한다.

### 🏐 httpBasic과 Bearer에 대하여

- 보통 서버의 경우 cookie를 http-only로만 허용하기 때문에 js를 통해 cookie를 서버로 넘겨주거나 하는 등의 방식은 지양하고 있다.
- 따라서 요청 헤더의 **Authorization 헤더에 유저의 credential을 담아 서버로 넘겨주어**서 인증을 하는 http Basic 방식을 사용할 수 있다.

![image-20210309230127654](../../post_images/20210309/image-20210309230127654.png)

- 하지만 이 경우, 패킷이 탈취당한다면 유저의 ID, Password가 그대로 노출이 되어 보안상 좋지 못한다.
- 따라서 **Authorization 헤더에 유저 credential대신 인증을 할 수 있는 token을 담아**서 보내어 인증을 하는 방식이 등장했는데 이 방식이 Bearer 방식이다.

![image-20210309230747911](../../post_images/20210309/image-20210309230747911.png)

- 토큰 또한 노출이 되지 않는 편이 좋겠지만 노출이 되더라도 직접적인 유저 credential이 보여지는 것이 아니며 만료시간이 존재하기 때문에 비교적 안전하다.

- 우리는 이 프로젝트에서 Bearer 방식을 사용할 것이기 때문에 SecurityConfig에서 httpBasic을 disable한 것이다.