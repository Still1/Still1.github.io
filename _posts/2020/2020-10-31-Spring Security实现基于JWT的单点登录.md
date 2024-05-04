---
title: Spring Security实现基于JWT的单点登录
tags: [Spring Security]
---

## Maven依赖

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
</dependency>

<dependency>  
    <groupId>io.jsonwebtoken</groupId>  
    <artifactId>jjwt</artifactId>  
</dependency>
```

0.9.1版本之后，对jjwt的引入如下

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.2</version>
</dependency>

<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
```

## JWT工具类

实现token的生成和解析

```java
@Component  
@AllArgsConstructor  
public class DoubucketJwtUtils {  
  
    private JwtProperties jwtProperties;  
  
    public String generateToken(String username) {  
        Integer validDurationHour = jwtProperties.getValidDurationHour();  
        Duration validDuration = Duration.ofHours(validDurationHour);  
        Date expireDate = new Date(System.currentTimeMillis() + validDuration.toMillis());  
        Date now = new Date();  
        String secret = jwtProperties.getSecret();  
        return Jwts.builder().setSubject(username).setIssuedAt(now)  
            .setExpiration(expireDate).signWith(SignatureAlgorithm.HS512, secret).compact();  
    }  
  
    public Claims parse(String token) {  
        Claims claims = null;  
        try {  
            claims = Jwts.parser().setSigningKey(jwtProperties.getSecret()).parseClaimsJws(token).getBody();  
        } catch (RuntimeException ignored) {  
        }  
        return claims;  
    }  
}
```

## Spring Security配置

开放`/getToken`的访问，增加JWT的鉴权认证

```java
@Configuration  
public class SecurityConfiguration {  
    @Bean  
    public SecurityFilterChain formFilterChain(HttpSecurity http) throws Exception {  
        http.authorizeRequests((requests) -> requests  
            .antMatchers(HttpMethod.POST, "/getToken").permitAll()  
            .anyRequest().authenticated())  
            .csrf().disable()  
            .apply(new JwtLoginConfigurer())  
            .and().sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);  
        return http.build();  
    }  
  
  
    @Bean  
    public UserDetailsService userDetailsService(UserService userService) {  
        return username -> {  
            User user = userService.getUserByUsername(username);  
            if (user == null) {  
                throw new UsernameNotFoundException("User:" + username + " not found");  
            }  
            return user;  
        };  
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();  
    }  
  
    @Bean  
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {  
        return authenticationConfiguration.getAuthenticationManager();  
    }  
}
```

上面对全局`AuthenticationManager`的获取可能受到Spring Security的版本影响，若获取失败，可使用下面的方法替代
{:.warning}

```java
@Bean
public AuthenticationManager authenticationManager(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
    DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
    daoAuthenticationProvider.setUserDetailsService(userDetailsService);
    daoAuthenticationProvider.setPasswordEncoder(passwordEncoder);
    return new ProviderManager(daoAuthenticationProvider);
}
```

实现token颁发

```java
@RestController  
@AllArgsConstructor  
public class LoginController {  
  
    private final LoginService loginService;  
  
    @PostMapping("getToken")  
    public String getToken(String username, String password) {  
        return loginService.getJwtToken(username, password);  
    }  
}
```

```java
@Service  
@AllArgsConstructor  
public class LoginServiceImpl implements LoginService {  
  
    private final AuthenticationManager authenticationManager;  
  
    private final DoubucketJwtUtils doubucketJwtUtils;  
  
    @Override  
    public String getJwtToken(String username, String password) {  
        if (StringUtils.isBlank(username) || StringUtils.isBlank(password)) {  
            throw new IllegalArgumentException();  
        }  
  
        UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken = new UsernamePasswordAuthenticationToken(username, password);  
        Authentication authenticate = authenticationManager.authenticate(usernamePasswordAuthenticationToken);  
        if (authenticate != null && authenticate.isAuthenticated()) {  
            return doubucketJwtUtils.generateToken(username);  
        } else {  
            throw new InvalidUserException();  
        }  
    }  
}
```

实现token验证

```java
public class JwtLoginConfigurer extends AbstractHttpConfigurer<JwtLoginConfigurer, HttpSecurity> {  
    @Override  
    public void configure(HttpSecurity builder) {  
        builder.authenticationProvider(new JwtAuthenticationProvider());  
  
        AuthenticationManager authenticationManager = builder.getSharedObject(AuthenticationManager.class);  
        builder.addFilterBefore(new JwtAuthenticationFilter(authenticationManager), BasicAuthenticationFilter.class);  
    }  
}
```

```java
@AllArgsConstructor  
public class JwtAuthenticationFilter extends OncePerRequestFilter {  
  
    private final AuthenticationManager authenticationManager;  
  
    @Override  
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
        JwtAuthenticationToken jwtAuthenticationToken = convertToToken(request);  
        if (jwtAuthenticationToken != null) {  
            try {  
                Authentication authenticate = authenticationManager.authenticate(jwtAuthenticationToken);  
                SecurityContext context = SecurityContextHolder.createEmptyContext();  
                context.setAuthentication(authenticate);  
                SecurityContextHolder.setContext(context);  
            } catch (AuthenticationException authenticationException) {  
                SecurityContextHolder.clearContext();  
            }  
        }  
        filterChain.doFilter(request, response);  
    }  
  
    private JwtAuthenticationToken convertToToken(HttpServletRequest request) {  
        String authorizationHeader = request.getHeader(HttpHeaders.AUTHORIZATION);  
        if (StringUtils.isBlank(authorizationHeader) || !StringUtils.startsWith(authorizationHeader, "Bearer ")) {  
            return null;  
        }  
  
        String token = authorizationHeader.split(" ")[1].trim();  
        return new JwtAuthenticationToken(token);  
    }  
}
```

```java
public class JwtAuthenticationToken extends AbstractAuthenticationToken {  
  
    private final String token;  
  
    private String principal;  
  
    public JwtAuthenticationToken(String token) {  
        super(null);  
        this.token = token;  
    }  
  
    @Override  
    public Object getCredentials() {  
        return "";  
    }  
  
    @Override  
    public Object getPrincipal() {  
        return principal;  
    }  
  
    public String getToken() {  
        return token;  
    }  
  
    public void setPrincipal(String principal) {  
        this.principal = principal;  
    }  
}
```

```java
public class JwtAuthenticationProvider implements AuthenticationProvider {  
    @Override  
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {  
        if (!(authentication instanceof JwtAuthenticationToken)) {  
            throw new IllegalArgumentException();  
        }  
  
        JwtAuthenticationToken jwtAuthenticationToken = (JwtAuthenticationToken) authentication;  
        String token = jwtAuthenticationToken.getToken();  
        DoubucketJwtUtils doubucketJwtUtils = ApplicationContextHolder.getBean(DoubucketJwtUtils.class);  
  
        Claims claims = doubucketJwtUtils.parse(token);  
        if (claims != null) {  
            String subject = claims.getSubject();  
            jwtAuthenticationToken.setPrincipal(subject);  
            jwtAuthenticationToken.setAuthenticated(true);  
            return jwtAuthenticationToken;  
        } else {  
            throw new InvalidJwtTokenException("invalid JWT token");  
        }  
    }  
  
    @Override  
    public boolean supports(Class<?> authentication) {  
        return (JwtAuthenticationToken.class.isAssignableFrom(authentication));  
    }  
}
```

## 参考文档

* [https://blog.csdn.net/u012760435/article/details/126587994?spm=1001.2014.3001.5502](https://blog.csdn.net/u012760435/article/details/126587994?spm=1001.2014.3001.5502)
* [https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)