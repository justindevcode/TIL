# todo

---
# 20250207
# JWT 임시세션, CORS 설정, JWT의 목표

## JWTFilter를 통과한 뒤 세션 확인
임시로 세션을 잠깐 만들기때문에 요청에 대해 세션에서 유저정보 꺼낼 수 있음  

```java
@Controller
@ResponseBody
public class MainController {

    @GetMapping("/")
    public String mainP() {

        String name = SecurityContextHolder.getContext().getAuthentication().getName();

        return "Main Controller : "+name;
    }
}
```

## 세션 현재 사용자 아이디
`SecurityContextHolder.getContext().getAuthentication().getName();`

## 세션 현재 사용자 role
```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
Iterator<? extends GrantedAuthority> iter = authorities.iterator();
GrantedAuthority auth = iter.next();
String role = auth.getAuthority();
```

## CORS 설정 자료
https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html  

## CORS란
https://ko.wikipedia.org/wiki/%EA%B5%90%EC%B0%A8_%EC%B6%9C%EC%B2%98_%EB%A6%AC%EC%86%8C%EC%8A%A4_%EA%B3%B5%EC%9C%A0  

## 발생 원리
![1](https://github.com/user-attachments/assets/0e3b479f-ebb6-4ecc-a8df-78d190bd05ad)  

## CORS 설정

* SecurityConfig
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		
		http
            .cors((corsCustomizer -> corsCustomizer.configurationSource(new CorsConfigurationSource() {

                @Override
                public CorsConfiguration getCorsConfiguration(HttpServletRequest request) {

                    CorsConfiguration configuration = new CorsConfiguration();

                    configuration.setAllowedOrigins(Collections.singletonList("http://localhost:3000"));
                    configuration.setAllowedMethods(Collections.singletonList("*"));
                    configuration.setAllowCredentials(true);
                    configuration.setAllowedHeaders(Collections.singletonList("*"));
                    configuration.setMaxAge(3600L);

										configuration.setExposedHeaders(Collections.singletonList("Authorization"));

                    return configuration;
                }
            })));

    return http.build();
}
```

* config>CorsMvcConfig
```java
@Configuration
public class CorsMvcConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry corsRegistry) {
        
        corsRegistry.addMapping("/**")
                .allowedOrigins("http://localhost:3000");
    }
}
```

## JWT를 깊게 생각하며 (강의자분이 생각하신 이유 메모한것)

JWT 공부를 시작했을 때 JWT를 왜 사용하는지를 잊어버려 한동안 고민에 잠겼고 결론을 찾아 헤매었던 적이 있습니다. 그 고민의 흐름과 주관적인 결론을 남깁니다.  

* JWT의 STATELESS 상태에 대한 집착
시큐리티 JWT config를 구성하며 STATELESS 상태에 초점이 맞추어졌습니다.  
(JWT 구현을 위해 STATELESS 상태가 필요하지만 STATELESS에 집착해야 하는건 아닙니다. 따라서 여기서 포인트 자체를 놓쳤습니다. 아무튼 이 과정을 작성하며 STATELESS 상태에 광적으로 집착을 하였습니다.)  

* 토큰 탈취의 문제 - Refresh 토큰의 도입과 Refresh 토큰도 동일한 상황
이때 부터 의문 사항이 생기게 되는 시발점입니다. JWT 자체가 세션 대비 토큰을 탈취 당했을때의 위험성이 큽니다.  
따라서 Refresh, Access라는 2가지의 토큰을 발급해주는데 Refresh 토큰 요청 주기 자체가 길기 때문에 탈취 당할 확률은 낮지만 탈취 당할 수 있다 입니다.  
이제 이 문제에 대한 여러 상황이 발생합니다.  

* 토큰이 탈취 되었을때 서버의 제어권과 로그아웃 문제 등등
토큰이 탈취되면 만료 기간 까지 서버측은 고통을 받습니다. 따라서 서버 비밀키를 변경하는 상황까지 도달하게 됩니다.  
프론트 서버측 로그아웃을 구현하여도 이미 토큰을 복제 했다면 계속 서버에 접속할 수 있기 때문에 여전히 문제가 있습니다.  
이를 위해 서버측 Redis와 같은 저장소에 발급한 Refresh 토큰을 저장한다는 구현들이 많았습니다. 그래서 로그아웃 상태거나 탈취된 토큰은 Redis 서버에서 제거하여 앞으로 Access 토큰 재발급이 불가능하도록 설정하는 것이었습니다.

* 깊은 고민의 발생 - 모순?
Refresh들을 저장하기 위한 Redis를 도입해버리면 사실상 세션 클러스터링을 작업하고 세션 방식을 사용하는 것이 좋지 않을까? STATELESS 작업을 했지만 다른 곳에서 상태 저장이 생겨버리네? (사실 엄밀하게는 아니지만 비슷한 맥락으로)  
아무튼 여기까지해서 많은 고민을 했고 탈취를 막으면서도 Redis를 도입하지 않을 방법에 대해서 한가지 방법을 떠올렸습니다.  

* IP 검증을 해보자 - 실패
처음 로그인이 요청된 IP를 JWT에 담아 매번 요청이 올때마다 JWT의 IP와 요청 IP가 동일한지 검증을 진행하는 방법을 구상했는데 사용자들의 단말기는 IP값이 동적으로 자주 변경되기 때문에 문제가 되어 실패했습니다.  

그래서 고민의 굴레에 빠졌습니다.  
STATELESS → 그런데 Redis → 그럼 차라리 세션 → 왜 JWT를 사용했지?  

## JWT를 왜 사용하는가?
* 독일의 철학자 한나 아렌트의 “만드는 사람과 만드는 동물” → 나는 만드는 동물인가?
위와 같이 꼬리에 꼬리를 무는 고민에 해답은 목표가 무엇인지 판단하는 것이었다.  
한나 아렌트에 의하면 인간은 어떤 일에 몰두하는 존재지만 동시에 판단하는 존재인 “만드는 사람”이다. 우리가 JWT의 목적을 확인하지 않고 구현에만 열중한다면 무엇을 하는지도 모르는 “만드는 동물”에 불과하다. 따라서 JWT의 STATELESS한 상태에만 목적을 두는 것이 아닌 JWT가 왜 필요한지를 생각했고 해답을 찾았다.

## JWT를 사용한 이유
* 모바일 앱
JWT가 사용된 주 이유는 결국 모바일 앱의 등장입니다. 모바일 앱의 특성상 주로 JWT 방식으로 인증/인가를 진행합니다. 결국 STATLESS는 부수적인 효과였습니다.  

* 모바일 앱에서의 로그아웃
모바일 앱에서는 JWT 탈취 우려가 거의 없기 때문에 앱단에서 로그아웃을 진행하여 JWT 자체를 제거해버리면 서버측에선 추가 조치도 필요가 없습니다. (토큰 자체가 확실하게 없어졌다는 보장이 되기 때문에)  

* 장시간 로그인과 세션
장기간 동안 로그인 상태를 유지하려고 세션 설정을 하면 서버 측 부하가 많이 가기 때문에 JWT 방식을 이용하는 것도 한 방법입니다.

## API 서버에서 Refresh 토큰을 저장하여 어느 정도의 상태(STATE)를 만드는 이유
이제는 상태를 남기는것에 대해서 큰 고민은 없어졌습니다. JWT의 목적이 STATELESS가 아니기 때문이니까요.  

## 참조
https://www.youtube.com/watch?v=Y1p6bVrRExs&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=12  
https://www.youtube.com/watch?v=MGkYFwdabeM&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=14  
https://www.youtube.com/watch?v=qIG-GNorXG4&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=15  

---
# 20250206
# 로그인 성공시 JWT 발급, JWT 검증 필터

## 로그인 성공
로그인 로직, JWTUtil 클래스를 생성하였습니다. 이제 로그인이 성공 했을 경우 JWT를 발급하기 위한 구현을 진행하겠습니다.  

## JWTUtil 주입

* LoginFilter : JWTUtil 주입
```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
		//JWTUtil 주입
		private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
				this.jwtUtil = jwtUtil;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

				//클라이언트 요청에서 username, password 추출
        String username = obtainUsername(request);
        String password = obtainPassword(request);

				//스프링 시큐리티에서 username과 password를 검증하기 위해서는 token에 담아야 함
        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

				//token에 담은 검증을 위한 AuthenticationManager로 전달
        return authenticationManager.authenticate(authToken);
    }

		//로그인 성공시 실행하는 메소드 (여기서 JWT를 발급하면 됨)
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

    }

		//로그인 실패시 실행하는 메소드
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

    }
}
```

* SecurityConfig에서 Filter에 JWTUtil 주입
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

		private final AuthenticationConfiguration authenticationConfiguration;
		//JWTUtil 주입
		private final JWTUtil jwtUtil;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration, JWTUtil jwtUtil) {

        this.authenticationConfiguration = authenticationConfiguration;
				this.jwtUtil = jwtUtil;
    }

		@Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {

        return configuration.getAuthenticationManager();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

				//AuthenticationManager()와 JWTUtil 인수 전달
        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration), jwtUtil), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

## LoginFilter 로그인 성공 successfulAuthentication 메소드 구현

* LoginFilter
```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {
				
				//UserDetailsS
        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();

        String username = customUserDetails.getUsername();

        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        Iterator<? extends GrantedAuthority> iterator = authorities.iterator();
        GrantedAuthority auth = iterator.next();

        String role = auth.getAuthority();

        String token = jwtUtil.createJwt(username, role, 60*60*10L);

        response.addHeader("Authorization", "Bearer " + token);
    }
}

```

HTTP 인증 방식은 RFC 7235 정의에 따라 아래 인증 헤더 형태를 가져야 한다.
```
Authorization: 타입 인증토큰

//예시
Authorization: Bearer 인증토큰string
```

## LoginFilter 로그인 실패 unsuccessfulAuthentication 메소드 구현
* LoginFilter
```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {
				
				//로그인 실패시 401 응답 코드 반환
        response.setStatus(401);
    }
}
```

## 발급 테스트
/login 경로로 username과 password를 포함한 POST 요청을 보낸 후 응답 헤더에서 Authorization 키에 담긴 JWT를 확인한다.  

* 요청 : POST /login : 아이디 비번 입력
* 응답 : return 헤더에 'Authorization' 키값으로 Bearer ~~~ 형식으로 리턴

## LoginFilter 최종
```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
        this.jwtUtil = jwtUtil;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

        String username = obtainUsername(request);
        String password = obtainPassword(request);

        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

        return authenticationManager.authenticate(authToken);
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();

        String username = customUserDetails.getUsername();

        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        Iterator<? extends GrantedAuthority> iterator = authorities.iterator();
        GrantedAuthority auth = iterator.next();

        String role = auth.getAuthority();

        String token = jwtUtil.createJwt(username, role, 60*60*10L);

        response.addHeader("Authorization", "Bearer " + token);
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

        response.setStatus(401);
    }
}
```

## JWT 검증 필터
스프링 시큐리티 filter chain에 요청에 담긴 JWT를 검증하기 위한 커스텀 필터를 등록해야 한다.  
해당 필터를 통해 요청 헤더 Authorization 키에 JWT가 존재하는 경우 JWT를 검증하고 강제로SecurityContextHolder에 세션을 생성한다. (이 세션은 STATLESS 상태로 관리되기 때문에 해당 요청이 끝나면 소멸 된다.)  

## JWTFilter 구현
* JWTFilter
```java
public class JWTFilter extends OncePerRequestFilter {

    private final JWTUtil jwtUtil;

    public JWTFilter(JWTUtil jwtUtil) {

        this.jwtUtil = jwtUtil;
    }


    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
				
				//request에서 Authorization 헤더를 찾음
        String authorization= request.getHeader("Authorization");
				
				//Authorization 헤더 검증
        if (authorization == null || !authorization.startsWith("Bearer ")) {

            System.out.println("token null");
            filterChain.doFilter(request, response);
						
						//조건이 해당되면 메소드 종료 (필수)
            return;
        }
			
        System.out.println("authorization now");
				//Bearer 부분 제거 후 순수 토큰만 획득
        String token = authorization.split(" ")[1];
			
				//토큰 소멸 시간 검증
        if (jwtUtil.isExpired(token)) {

            System.out.println("token expired");
            filterChain.doFilter(request, response);

						//조건이 해당되면 메소드 종료 (필수)
            return;
        }

				//토큰에서 username과 role 획득
        String username = jwtUtil.getUsername(token);
        String role = jwtUtil.getRole(token);
				
				//userEntity를 생성하여 값 set
        UserEntity userEntity = new UserEntity();
        userEntity.setUsername(username);
        userEntity.setPassword("temppassword");
        userEntity.setRole(role);
				
				//UserDetails에 회원 정보 객체 담기
        CustomUserDetails customUserDetails = new CustomUserDetails(userEntity);

				//스프링 시큐리티 인증 토큰 생성
        Authentication authToken = new UsernamePasswordAuthenticationToken(customUserDetails, null, customUserDetails.getAuthorities());
				//세션에 사용자 등록
        SecurityContextHolder.getContext().setAuthentication(authToken);

        filterChain.doFilter(request, response);
    }
}
```

## SecurityConfig JWTFilter 등록
* SecurityConfig
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final AuthenticationConfiguration authenticationConfiguration;
    private final JWTUtil jwtUtil;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration, JWTUtil jwtUtil) {

        this.authenticationConfiguration = authenticationConfiguration;
        this.jwtUtil = jwtUtil;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());
				
				//JWTFilter 등록
        http
                .addFilterBefore(new JWTFilter(jwtUtil), LoginFilter.class);

        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration), jwtUtil), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

## JWT 요청 인가 테스트
요청 헤더에 JWT를 첨부하고 로그인이 권한이 필요한 페이지에 접근을 진행하겠습니다.  

* 요청 : 헤더에 로그인시 받은 'Authorization' 키값으로 Bearer ~~~ 형식 사용해 요청
* 응답 : 접속 성공

## 참조
https://www.youtube.com/watch?v=nH3A22izjY0&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=10  
https://www.youtube.com/watch?v=7B6KHSZN3jY&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=12  

---
# 20250114
# DB기반 로그인검증로직, JWT 발급 클레스

## DB 기반 로그인 검증

![1](https://github.com/user-attachments/assets/98aa414d-5d1d-480e-9c3f-8e02b76ea4ea)  

지난번 AuthenticationManager 앞단을 구현했고 이번 8강에서 DB에서 AuthenticationManager까지 로직을 구현하겠습니다.  
구현은 UserDetails, UserDetailsService, UserRepository의 회원 조회 메소드를 진행하겠습니다.  

## UserRepository

* UserRepository
```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

    Boolean existsByUsername(String username);
		
		//username을 받아 DB 테이블에서 회원을 조회하는 메소드 작성
    UserEntity findByUsername(String username);
}
```

## UserDetailsService 커스텀 구현
* CustomUserDetailsService
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {

        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
				
				//DB에서 조회
        UserEntity userData = userRepository.findByUsername(username);

        if (userData != null) {
						
						//UserDetails에 담아서 return하면 AutneticationManager가 검증 함
            return new CustomUserDetails(userData);
        }

        return null;
    }
}
```

## UserDetails 커스텀 구현
* CustomUserDetails
```java
public class CustomUserDetails implements UserDetails {

    private final UserEntity userEntity;

    public CustomUserDetails(UserEntity userEntity) {

        this.userEntity = userEntity;
    }


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {

        Collection<GrantedAuthority> collection = new ArrayList<>();

        collection.add(new GrantedAuthority() {

            @Override
            public String getAuthority() {

                return userEntity.getRole();
            }
        });

        return collection;
    }

    @Override
    public String getPassword() {

        return userEntity.getPassword();
    }

    @Override
    public String getUsername() {

        return userEntity.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {

        return true;
    }

    @Override
    public boolean isAccountNonLocked() {

        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {

        return true;
    }

    @Override
    public boolean isEnabled() {

        return true;
    }
}
```
우리가 형식 맞춰서 만들어줘야함  

## JWT 발급과 검증

* 로그인시 → 성공 → JWT 발급
* 접근시 → JWT 검증

JWT에 관해 발급과 검증을 담당할 클래스가 필요하다. 따라서 JWTUtil이라는 클래스를 생성하여 JWT 발급, 검증 메소드를 작성하는 시간입니다.  

## JWT 생성 원리

공식페이지 : https://jwt.io/  

JWT는 Header.Payload.Signature 구조로 이루어져 있다. 각 요소는 다음 기능을 수행한다.  

* Header
	* JWT임을 명시
	* 사용된 암호화 알고리즘
* Payload
	* 정보
* Signature
	* 암호화알고리즘((BASE64(Header))+(BASE64(Payload)) + 암호화키)


JWT의 특징은 내부 정보를 단순 BASE64 방식으로 인코딩하기 때문에 외부에서 쉽게 디코딩 할 수 있다.  
외부에서 열람해도 되는 정보를 담아야하며, 토큰 자체의 발급처를 확인하기 위해서 사용한다.  
(지폐와 같이 외부에서 그 금액을 확인하고 금방 외형을 따라서 만들 수 있지만 발급처에 대한 보장 및 검증은 확실하게 해야하는 경우에 사용한다. 따라서 토큰 내부에 비밀번호와 같은 값 입력 금지)  

## JWT 암호화 방식

* 암호화 종류
	* 양방향
		* 대칭키 - 이 프로젝트는 양방향 대칭키 방식 사용 : HS256
		* 비대칭키
	* 단방향

## 암호화 키 저장

암호화 키는 하드코딩 방식으로 구현 내부에 탑재하는 것을 지양하기 때문에 변수 설정 파일에 저장한다.  

```
spring.jwt.secret=vmfhaltmskdlstkfkdgodyroqkfwkdbalroqkfwkdbalaaaaaaaaaaaaaaaabbbbb
```

## JWTUtil

* 토큰 Payload에 저장될 정보
	* username
	* role
	* 생성일
	* 만료일
* JWTUtil 구현 메소드
	* JWTUtil 생성자
	* username 확인 메소드
	* role 확인 메소드
	* 만료일 확인 메소드

* JWTUtil : 0.12.3
```java
@Component
public class JWTUtil {

    private SecretKey secretKey;

    public JWTUtil(@Value("${spring.jwt.secret}")String secret) {


        secretKey = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), Jwts.SIG.HS256.key().build().getAlgorithm());
    }

    public String getUsername(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().get("username", String.class);
    }

    public String getRole(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().get("role", String.class);
    }

    public Boolean isExpired(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().getExpiration().before(new Date());
    }

    public String createJwt(String username, String role, Long expiredMs) {

        return Jwts.builder()
                .claim("username", username)
                .claim("role", role)
                .issuedAt(new Date(System.currentTimeMillis()))
                .expiration(new Date(System.currentTimeMillis() + expiredMs))
                .signWith(secretKey)
                .compact();
    }
}
```
정보 가저오는 방식 버전마다 좀 다름, 암호화키 사용법도 정해진방식 맞춰서 사용해줘야함  


* JWTUtil : 0.11.5
```java
@Component
public class JWTUtil {

    private Key key;

    public JWTUtil(@Value("${spring.jwt.secret}")String secret) {


				byte[] byteSecretKey = Decoders.BASE64.decode(secret);
        key = Keys.hmacShaKeyFor(byteSecretKey);
    }

    public String getUsername(String token) {

        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody().get("username", String.class);
    }

    public String getRole(String token) {

        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody().get("role", String.class);
    }

    public Boolean isExpired(String token) {

        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody().getExpiration().before(new Date());
    }

    public String createJwt(String username, String role, Long expiredMs) {

				Claims claims = Jwts.claims();
        claims.put("username", username);
        claims.put("role", role);

        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + expiredMs))
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }
}
```

## 참조
https://www.youtube.com/watch?v=q4eTHvQAvRo&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=8  
https://www.youtube.com/watch?v=obNHwsl0fXM&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=10  

---
# 20250113
# 회원가입 로직구현, 로그인 필더 구현

## 회원가입 로직

![1](https://github.com/user-attachments/assets/eeb1d86a-eb16-4c2c-9a18-1e5f28dd98b1)  

## JoinDTO
* JoinDTO
```java
@Setter
@Getter
public class JoinDTO {

    private String username;
    private String password;
}
```

## JoinController

* JoinController
```java
@Controller
@ResponseBody
public class JoinController {
    
    private final JoinService joinService;

    public JoinController(JoinService joinService) {
        
        this.joinService = joinService;
    }

    @PostMapping("/join")
    public String joinProcess(JoinDTO joinDTO) {

        System.out.println(joinDTO.getUsername());
        joinService.joinProcess(joinDTO);

        return "ok";
    }
}
```

## JoinService
* JoinService
```java
@Service
public class JoinService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;

    public JoinService(UserRepository userRepository, BCryptPasswordEncoder bCryptPasswordEncoder) {

        this.userRepository = userRepository;
        this.bCryptPasswordEncoder = bCryptPasswordEncoder;
    }

    public void joinProcess(JoinDTO joinDTO) {

        String username = joinDTO.getUsername();
        String password = joinDTO.getPassword();

        Boolean isExist = userRepository.existsByUsername(username);

        if (isExist) {

            return;
        }

        UserEntity data = new UserEntity();

        data.setUsername(username);
        data.setPassword(bCryptPasswordEncoder.encode(password));
        data.setRole("ROLE_ADMIN");

        userRepository.save(data);
    }
}
```

## UserRepository

* UserRepository
```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

    Boolean existsByUsername(String username);
}
```

## 로그인 필터 구현 자료

* 스프링 시큐리티 필터 참고 자료 : https://docs.spring.io/spring-security/reference/servlet/architecture.html

## 로그인 모식도
![1](https://github.com/user-attachments/assets/c6799304-e961-4ca9-aa10-d89251c380c5)  

## 스프링 시큐리티 필터 동작 원리
스프링 시큐리티는 클라이언트의 요청이 여러개의 필터를 거쳐 DispatcherServlet(Controller)으로 향하는 중간 필터에서 요청을 가로챈 후 검증(인증/인가)을 진행한다.  

* 클라이언트 요청 → 서블릿 필터 → 서블릿 (컨트롤러)
<img width="530" alt="1" src="https://github.com/user-attachments/assets/0ad528bd-103d-47cd-ae2e-23dbf5e856a3" />

* Delegating Filter Proxy
서블릿 컨테이너 (톰캣)에 존재하는 필터 체인에 DelegatingFilter를 등록한 뒤 모든 요청을 가로챈다.

<img width="290" alt="1" src="https://github.com/user-attachments/assets/1b4daa96-dbc2-4597-bb96-f8b852a22d73" />  

* 서블릿 필터 체인의 DelegatingFilter → Security 필터 체인 (내부 처리 후) → 서블릿 필터 체인의 DelegatingFilter
가로챈 요청은 SecurityFilterChain에서 처리 후 상황에 따른 거부, 리디렉션, 서블릿으로 요청 전달을 진행한다.

<img width="625" alt="1" src="https://github.com/user-attachments/assets/883b599f-7f16-4aee-ae2e-55163356f736" />  

* SecurityFilterChain의 필터 목록과 순서
<img width="290" alt="1" src="https://github.com/user-attachments/assets/90ea5e9c-aed7-40bd-95ae-6aed6c0055ba" />

(모든 필터는 전부 활성화되지 않습니다.)  

## Form 로그인 방식에서 UsernamePasswordAuthenticationFilter
Form 로그인 방식에서는 클라이언트단이 username과 password를 전송한 뒤 Security 필터를 통과하는데 UsernamePasswordAuthentication 필터에서 회원 검증을 진행을 시작한다.  
(회원 검증의 경우 UsernamePasswordAuthenticationFilter가 호출한 AuthenticationManager를 통해 진행하며 DB에서 조회한 데이터를 UserDetailsService를 통해 받음)  
우리의 JWT 프로젝트는 SecurityConfig에서 formLogin 방식을 disable 했기 때문에 기본적으로 활성화 되어 있는 해당 필터는 동작하지 않는다.  
따라서 로그인을 진행하기 위해서 필터를 커스텀하여 등록해야 한다.  

## 로그인 로직 구현 목표

* 아이디, 비밀번호 검증을 위한 커스텀 필터 작성
* DB에 저장되어 있는 회원 정보를 기반으로 검증할 로직 작성
* 로그인 성공시 JWT를 반환할 success 핸들러 생성
* 커스텀 필터 SecurityConfig에 등록


## 로그인 요청 받기 : 커스텀 UsernamePasswordAuthentication 필터 작성
로그인 검증을 위한 커스텀 UsernamePasswordAuthentication 필터 작성  

* LoginFilter
```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;

    public LoginFilter(AuthenticationManager authenticationManager) {

        this.authenticationManager = authenticationManager;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

				//클라이언트 요청에서 username, password 추출
        String username = obtainUsername(request);
        String password = obtainPassword(request);

				//스프링 시큐리티에서 username과 password를 검증하기 위해서는 token에 담아야 함 (null 매개변수쪽에는 원래 권한 담겨야함)
        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

				//token에 담은 검증을 위한 AuthenticationManager로 전달
        return authenticationManager.authenticate(authToken);
    }

		//로그인 성공시 실행하는 메소드 (여기서 JWT를 발급하면 됨)
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

    }

		//로그인 실패시 실행하는 메소드
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

    }
}
```

## SecurityConfig 설정
* SecurityConfig : 커스텀 로그인 필터 등록
```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

				//필터 추가 LoginFilter()는 인자를 받음 (AuthenticationManager() 메소드에 authenticationConfiguration 객체를 넣어야 함) 따라서 등록 필요
        http
                .addFilterAt(new LoginFilter(), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

* SecurityConfig : AuthenticationMananger Bean 등록과 LoginFilter 인수 전달
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    //AuthenticationManager가 인자로 받을 AuthenticationConfiguraion 객체 생성자 주입
    private final AuthenticationConfiguration authenticationConfiguration;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration) {

        this.authenticationConfiguration = authenticationConfiguration;
    }

    //AuthenticationManager Bean 등록
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {

        return configuration.getAuthenticationManager();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

//필터 추가 LoginFilter()는 인자를 받음 (AuthenticationManager() 메소드에 authenticationConfiguration 객체를 넣어야 함) 따라서 등록 필요
        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration)), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

## 로그인 성공시 JWT 반환
로그인 성공시 successfulAuthentication() 메소드를 통해 JWT를 응답해야 한다. 따라서 JWT 응답 구문을 작성해야 하는데 JWT 발급 클래스를 아직 생성하지 않았기 때문에 다음 시간에 DB 기반 회원 검증 구현을 진행한 뒤 JWT 발급 및 검증을 진행하는 클래스를 생성하겠습니다.  

## 참조
https://www.youtube.com/watch?v=yNACbJF4uoo&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=6  
https://www.youtube.com/watch?v=3Ff7UHGG3t8&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=8  

---
# 20250107
# 시큐리티config, entity, DB설정

## SecurityConfig 클래스 설명
스프링 시큐리티의 인가 및 설정을 담당하는 클래스이다. Security Config 구현은 스프링 시큐리티의 세부 버전별로 많이 상이합니다. (이번 시리즈는 스프링 시큐리티 6.2.1 버전으로 구현합니다.)  
자주 접할 수 있는 버전에 대한 구현 차이는 아래 영상을 통해 확인할 수 있습니다.  

* 스프링 시큐리티 시리즈 : 버전별 Security Config 구현 방법
https://www.youtube.com/watch?v=NdRVhOccuOs

## Security Config 클래스 기본 요소 작성
시큐리티 JWT 구현을 위한 Config 클래스의 일부분을 작성할 예정입니다. 먼저 기본적인 설정만 진행하고 시리즈를 진행하며 커스텀 필터 요소들을 추가 구현할 예정입니다.  

* SecurityConfig
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

				//csrf disable
        http
                .csrf((auth) -> auth.disable());

				//From 로그인 방식 disable
        http
                .formLogin((auth) -> auth.disable());

				//http basic 인증 방식 disable
        http
                .httpBasic((auth) -> auth.disable());

				//경로별 인가 작업
        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
												.requestMatchers("/admin").hasRole("ADMIN")
                        .anyRequest().authenticated());

				//세션 설정
        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```
JWT를 통한 인증/인가를 위해서 세션을 STATELESS 상태로 설정하는 것이 중요하다.  

## BCryptPaasswordEncoder 등록
* SecurityConfig
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

## POSTMAN 설치
API 서버는 웹서버와 달리 서버측으로 요청을 보낼 수 있는 페이지가 존재하지 않고 엔드 포인트만 존재하기 때문에 요청을 보낼 API 클라이언트가 필요하다.  

* 공식 홈페이지 주소
https://www.postman.com/downloads/

## DB연결 및 Entity 작성, 데이터베이스 종류와 ORM
회원 정보를 저장하기 위한 데이터베이스는 MySQL 엔진의 데이터베이스를 사용한다. 그리고 접근은 Spring Data JPA를 사용한다.  

## 데이터베이스 의존성 주석 해제
2강에서 진행했던 build.gradle의 Spring Data JPA 및 MySQL Driver 의존성 주석을 해제한다.  

## 변수 설정
* DB 연결 설정 : application.properteis
```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://아이피:3306/데이터베이스?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul&allowPublicKeyRetrieval=true
spring.datasource.username=아이디
spring.datasource.password=비밀번호
```

* Hibernate ddl 설정 : application.properites
```
spring.jpa.hibernate.ddl-auto=none
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

## DB 연결에 대한 자세한 설명 참고
https://www.youtube.com/watch?v=7dhbaMWaJ3Y  

## 회원 테이블 Entity 작성 : UserEntity
* UserEntity
```java
@Entity
@Setter
@Getter
public class UserEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String username;
    private String password;

    private String role;
}
```

## 회원 테이블 Repository 작성 : UserRepository
* UserRepository
```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

}
```

## ddl-auto=create 설정 후 실행
데이터베이스에서 회원 정보를 저장할 테이블을 생성해야 하지만 ddl-auto 설정을 통해 스프링 부트 Entity 클래스 기반으로 테이블을 생성할 수 있다.  

## 참고
https://www.youtube.com/watch?v=A3YsWHGbeZQ&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=3  
https://www.youtube.com/watch?v=TN2rEgvQObM&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=5  
https://www.youtube.com/watch?v=JFTpzy7gsg0&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=6  

---
# 20250106
# 스프링 시큐리티 JWT 동작원리와 프로젝트 생성

## 실습 목표
스프링 시큐리티 6 프레임워크를 활용하여 JWT 기반의 인증/인가를 구현하고 회원 정보 저장(영속성) MySQL 데이터베이스를 활용한다.  
서버는 API 서버 형태로 구축한다. (웹 페이지를 응답하는 것이 아닌 API 클라이언트 요청을 통해 데이터 응답만 확인함)  

## 구현
* 인증 : 로그인
* 인가 : JWT를 통한 경로별 접근 권한
* 회원가입

## JWT 인증 방식 시큐리티 동작 원리

* 회원가입 : 내부 회원 가입 로직은 세션 방식과 JWT 방식의 차이가 없다.

![1](https://github.com/user-attachments/assets/22d99417-28aa-4a97-b789-28d00dda1d01)  

* 로그인 (인증) : 로그인 요청을 받은 후 세션 방식은 서버 세션이 유저 정보를 저장하지만 JWT 방식은 토큰을 생성하여 응답한다.

![1](https://github.com/user-attachments/assets/253cf3b7-a90b-4fa2-b433-cc3831f95060)  

* 경로 접근 (인가) : JWT Filter를 통해 요청의 헤더에서 JWT를 찾아 검증을하고 일시적 요청에 대한 Session을 생성한다. (생성된 세션은 요청이 끝나면 소멸됨)

![1](https://github.com/user-attachments/assets/7acf974e-c9a0-4f9d-83a6-52f863d10e42)  

## 버전 및 의존성

* Spring Boot 3.2.1
* Security 6.2.1
* Lombok
* Spring Data JPA - MySQL
* Gradle - Groovy
* IntelliJ Ultimate

## 기타
* 스프링 시큐리티 JWT 구현 방법이 아주 많습니다. 개발자별 다른 구현을 진행하고 버전별로도 메소드가 많이 다릅니다. 최대한 공식 문서에 구현된 형태로 코드를 작성하지만 구현은 다를 수 있습니다.
 
* 저는 간단하게 API 서버에서 JWT 구현을 진행했고 토큰 발급의 경우 단일 토큰으로 진행합니다. (Access, Refresh로 나누는 경우도 있지만 기본 강의라 간단하게 한개로 진행하겠습니다.)
 

* 스프링 시큐리티 JWT 시리즈를 끝낸 후 OAuth2 소셜 로그인도 진행하려 합니다. (따라서 이번 시리즈에서는 소셜 로그인을 진행하지 않습니다.)
 

* 개념적인 부분 설명은 어디서든지 찾아볼 수 있기 때문에 많이 줄이고 구현 실습 위주로 진행하겠습니다.
 

* 제 코드는 JWT를 구현하기 위한 가장 기본적인 뼈대 코드입니다.

공식문서 : https://docs.spring.io/spring-security/reference/servlet/architecture.html  

## 프로젝트 생성 및 의존성 추가

필수 의존성
* Lombok
* Spring Web
* Spring Security
* Spring Data JPA
* MySQL Driver

## 데이터베이스 의존성 주석 처리
임시로 주석 처리 진행 (스프링 부트에서 데이터베이스 의존성을 추가한 뒤 연결을 진행하지 않을 경우 런타임 에러 발생)  

## JWT 필수 의존성
JWT 토큰을 생성하고 관리하기 위해 JWT 의존성을 필수적으로 설정해야 합니다.  
설정은 build.gradle을 통해 진행하며 이때 버전을 선택하여 적용해야 합니다.  

대부분은 JWT 0.11.5 버전을 통해 구현하지만 최신 버전은 0.12.3입니다. 따라서 0.12.3을 기반으로 구현하지만 추가적으로 0.11.5 버전에 대한 구현 방법도 올릴 예정입니다. (JWT를 생성하고 내부에서 데이터를 얻는 메소드가 버전마다 많이 상이합니다.)  

* 0.12.3 버전 : build.gradle
```
dependencies {

    implementation 'io.jsonwebtoken:jjwt-api:0.12.3'
    implementation 'io.jsonwebtoken:jjwt-impl:0.12.3'
    implementation 'io.jsonwebtoken:jjwt-jackson:0.12.3'
}
```

* 0.11.5 버전 : build.gradle
```
dependencies {

    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    implementation 'io.jsonwebtoken:jjwt-impl:0.11.5'
    implementation 'io.jsonwebtoken:jjwt-jackson:0.11.5'
}
```

* 의존성 BOM
https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api/0.12.3

https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api/0.11.5  

## 기본 Controller 생성
* MainController
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
public class MainController {

    @GetMapping("/")
    public String mainP() {

        return "main Controller";
    }
}
```

* AdminController
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@ResponseBody
public class AdminController {

    @GetMapping("/admin")
    public String adminP() {

        return "admin Controller";
    }
}
```

## 참고
https://www.youtube.com/watch?v=NPRh2v7PTZg&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ  
https://www.youtube.com/watch?v=ZTaZOCqTez4&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=3  

---
# 20250102
# 윈도우즈 환경 WSL 설치, gradle 네이티브 컴파일 방법

## 네이티브 빌드 스펙
스프링을 네이티브로 빌드하기 위해선 까다로운 스펙이 요구됩니다.  

* 소프트웨어 스펙
우선 윈도우즈 환경은 구성 가능하지만 설정할 부분이 매우 많고 안돌아가는 경우도 많기 때문에 리눅스 환경을 99% 추천 드립니다.  
이번 시리즈에서는 우분투 22.04 환경을 사용하도록 하겠습니다. (버전은 크게 상관 없습니다.)  

* 하드웨어 스펙
AWS EC2 환경을 체택하려 했으나 t2.micro ~ t3.large 환경까지 테스트한 결과 CPU 부하로 인해 빌드가 모두 실패했습니다.  
I7-13세대 환경에서도 CPU 부하율이 1분간 100%를 차지하기 때문에 상당히 좋은 수준의 CPU가 요구됩니다.  
대략적인 테스트 결과 램은 최소 8GB 이상, CPU는 I3-13세대 이상의 성능을 추천드립니다.  
(매우 안좋은 최적화 상태를 보입니다.)

## WSL 필요
대부분의 시청자분들은 윈도우즈 환경을 사용하실거라 예상합니다. 빌드를 위해 리눅스를 준비해야하는데 클라우드 환경은 실패하는 경우가 많기 때문에 윈도우즈 환경 위에 가상화 리눅스를 올리는 WSL을 통해 우분투 22.04를 설치하도록 하겠습니다.  
성능 좋은 리눅스나 맥 환경을 소유하신 시청자분들은 WSL 설치 과정이 필요 없습니다.  
!!!! 무조건 윈도우즈 프로 버전이 필요합니다, 홈 에디션 설치 불가!!!!  

## 윈도우즈 기본 설정
* Hyper-V 활성화 확인 : 가상화 : 사용
* Linux용 Windows 하위 시스템 활성 (윈도우즈 검색에서 : "Windows 기능 켜기/끄기" 검색 > 제어판 → 프로그램 → "Windows 기능 켜기/끄기")

## WSL 설치

* POWERSHELL 관리자 모드 실행
* 설치 명령어 입력
`wsl --install`

## WSL 명령어 안먹는 경우
* CMD 관리자 권한 실행
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

* 재부팅

* POWERSHELL 관리자 권한 실행
`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`

시간 어느정도 소요

* 재부팅

## WSL에 우분투 설치
* 리눅스 목록 확인
`wsl --list --online`  

* 우분투 22.04 버전 설치
`wsl --install Ubuntu-22.04`  

* 아이디입력
* 비밀번호 2회 입력 (없으면 엔터키)

## 접속
* 설치된 목록 확인
`wsl -l -v`

* 접속
`wsl -d Ubuntu-22.04`

## 설치된 리눅스 제거
`wsl --unregister Ubuntu-22.04`

## gradle 네이티브 컴파일 방법

GraalVM Native Support 의존성을 추가한 뒤 스프링 프로젝트를 만드셨다면, 네이티브로 빌드를 진행할 수 있습니다.  

## 네이티브 빌드 방식

설명드린 내용처럼 2가지 방법으로 빌드할 수 있고 각 방식의 과정과 결과는 아래와 같습니다.  

* nativeCompile : 실행 파일로 빌드하는 방식
* bootBuildImage : 실행 파일을 포함한 도커 이미지로 빌드하는 방식

두 방법에 대해서 기본적인 리눅스 빌드 환경을 구성하는 방식은 거의 동일하며 bootBuildImage의 경우에만 후반부에 추가적으로 build.gradle 및 docker 설정 부분이 요구됩니다.  

## 프로젝트를 리눅스 환경으로 이동
방법1 : git repository에 업로드 후 다운로드 방식  
방법2 : scp 명령어를 통한 전송  

이후 바로 빌드 명령어인 아래 명령을 입력하면 환경 구성이 없기 때문에 실행되지 않습니다.  
```
./gradlew nativeCompile
./gradlew bootBuildImage
```

## 리눅스 빌드 환경 구성

초반부는 natvieCompile과 bootBuildImage가 동일합니다.  

* apt 업데이트 및 기본적인 gcc 설치
```
sudo apt update
sudo apt install build-essential zlib1g-dev unzip -y
```

* graalVM 환경 설치
(공식 설치 링크 : https://www.graalvm.org/downloads/#)
```
wget https://download.oracle.com/graalvm/22/latest/graalvm-jdk-22_linux-x64_bin.tar.gz

sudo tar -xzf graalvm-jdk-22_linux-x64_bin.tar.gz
sudo mv graalvm-jdk-22.0.2+9.1 /usr/lib/graalvm
```
1. wget 또는 curl로 다운로드  
2. 다운로드한 파일 tar로 압축 해제  
3. 압축해제한 폴더를 /usr/lib/ 경로로 이동

* Java 환경 변수 설정
java 명령어를 사용하여 graalVM을 부를 수 있도록 환경 변수 설정 (상단에서 3번에 이동한 경로로 설정 진행)  
```
export GRAALVM_HOME=/usr/lib/graalvm
export JAVA_HOME=$GRAALVM_HOME
export PATH=$JAVA_HOME/bin:$PATH

java -version
native-image --version
```
(우리는 native-image를 통해 스프링 프로젝트를 네이티브 빌드하기 때문에 같이 설치가 되었는지 꼭 확인이 필요합니다. 과거 버전이 경우는 gu라는 JDK 설치 도구를 통해 추가 설치를 해야하지만 현재는 포함되어 있습니다.)  

## ./gradlew nativeCompile
실행 파일로 구성하는 방법  
우리의 스프링 프로젝트 경로로 진입 후 gradlew 파일이 존재하는 곳에서  

* 빌드 진행
`./gradlew nativeCompile`

* 결과
프로젝트 내부 아래 경로에서 확인 가능
`/build/native/nativeCompile`

## ./gradlew bootBuildImage
실행 파일을 포함한 도커 이미지로 구성하는 방법  
여기서는 추가로 build.gradle 값 추가와 우분투에 Docker 환경을 설치하셔야 합니다.  

* build.gradle 명령어 추가
```
bootBuildImage {
    imageName = 'nativetest'
}
```
소문자만 지원되며 빌드 후 도커 이미지 이름 값으로 부여됩니다.  

* Docker 설치
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt install docker-ce

sudo systemctl status docker

```

* Docker 소켓에 권한 부여 (gradlew에서 접근을 위함)
`sudo chmod 777 /var/run/docker.sock`

* 빌드 진행
`./gradlew bootBuildImage`

* 결과
`docker images`

## 참고
https://www.youtube.com/watch?v=5tDddRjFNeo&list=PLJkjrxxiBSFAvGAVh6mYpGBLr_h0wqLnV&index=3  
https://www.youtube.com/watch?v=yncBMm85sa4&list=PLJkjrxxiBSFAvGAVh6mYpGBLr_h0wqLnV&index=5  

---
# 20241231
# 스프링 네이티브 GraalVM 기초 설명과 특징확인

## 목표
스프링 네이티브는 최근 출시된 기능 (스프링 부트3 ~)으로 MSA 시대를 맞이하여 준비한다면 하나의 솔루션으로 사용하기 좋은 기능입니다.  
꼭 MSA 환경이 아니라도 구성 및 동작은 가능하기 때문에 실습 정도는 좋을거 같습니다.  
(스프링 네이티브 도구는 음정이 없는 타악기 같은 존재로 그 자체로도 공연이 가능하지만 기타 현악기나 건반악기와 어울리면 더 좋은 화합을 낼 수 있는 그런 친구입니다.)  

## Spring Native란?
쉽게 말해 스프링을 실행 파일로 배포해 JVM 환경 없이 바로 실행 시킬 수 있도록 만드는 방법입니다.  
AOT, JIT 여러가지 아키텍처를 알아야 하는 부분이 많은데 결과적으론 jar 파일 대비 실행 시간이 기하급수적으로 줄어듭니다.  
jar 파일은 아이들까지 20초 걸리는 상황이라면 실행 파일은 1초만에 배포되기 때문에 빠른 배포가 불가능하지만 네이티브는 가능하기 때문에 아래와 같은 이점이 있습니다.  

## 스프링 네이티브로 가져올 수 있는 이점
MSA를 구성하면서 보통 하나의 도메인에 대한 컨테이너 이미지를 만들고 트래픽의 크기에 따른 오토스케일링을 진행합니다.  

![1](https://github.com/user-attachments/assets/d165a2f7-cdd2-4948-906c-9bec17361215)  
급격하게 많은 트래픽이 쏟아질 경우 컨테이너 이미지를 통해 추가 인스턴스를 열어야하는데 jar기반 컨테이너의 경우 아이들까지의 실행시간이 존재하기 때문에 네이티브 기반 컨테이너로 교체할 경우 급격한 트래픽을 받아 서비스를 안전하게 운용할 수 있습니다.  

즉 스프링 실행후 포트번호가 뜨면서 프로그램이 완전히 올라갔다는 로그가 뜰떄까지 0.2초면 끝납니다.  

## 스프링 네이티브 단점
추후에 알려드릴 AOT, JIT이라는 컴파일러 개념이 있습니다.  
기존의 경우 JIT 방식으로 런타임에서 코드 최적화를 통한 컴파일을 수행하는 방식인데 네이티브 이미지의 경우 이 방식이 불가능하기 때문에 동적과 관련된 여러 라이브러리 사용이 불가능합니다.  

aop 스프링 프로필 등등 대부분의 굵직한 기능을 사용하지못하고 런타임중 최적화등의 기능도 작동하지 않게됩니다.  

## 실습 환경 및 시연
강의자님이 3주간 테스트하고 reflect-config.json도 다 뜯었는데, 안되는 케이스가 너무 많습니다.  
되도록 사용하지 않는 것을 추천드립니다.  

## 의존성
* 자바 : 17
* 빌드도구 : gradle
* 필수 의존성 : GraalVM Native Support
* 선택 의존성 : Lombok, Spring Web, Spring Security, Spring Data JPA, MySQL Driver

## GraalVM 환경
자바 프로그램을 가동하기 위해선 JVM 환경이 요구됩니다.  
JVM을 제공하는 여러 기업이 있지만 그 중 Oracle에서 Oracle JDK와 별개로 신기술을 적용한 JVM인 GraalVM을 제공하고 있습니다.  
네이티브 환경의 스프링을 배포하기 위해선 GraalVM 환경이 필수적으로 요구되기 때문에 GraalVM 환경을 구성하셔야 합니다.  
개발 환경의 경우 필수가 아니지만 네이티브로 빌드하기 위한 환경은 필수적으로 요구 됩니다.  
이번 시리즈 빌드용 GraalVM 버전은 아래와 같습니다.  
`graalvm 22.0.2`  

## 프로젝트 작성과 주의점
GraalVM 네이티브 이미지 지원 관련 문서 : https://docs.spring.io/spring-boot/reference/packaging/native-image/introducing-graalvm-native-images.html  

기존 JDK 환경에서 동작하던 스프링의 모든 의존성과 구현이 네이티브 빌딩 과정에서는 안되는 경우가 있습니다.  
네이티브 빌드를 위해 특정 설정이 필요한데 현재 스프링의 메이저한 의존성들은 자동 설정이 되지만 마이너한 의존성의 경우 안되는 경우가 많고, @Profile과 같은 어노테이션 사용이 불가능합니다.  

## 네이티브 빌드 방법
Gradle 빌드 환경에서 네이티브로 빌드 할 수 있는 방법은 2가지가 존재합니다.  

* nativeCompile : 실행 파일로 빌드하는 방법
* bootBuildImage : 실행 파일을 포함한 도커 이미지로 빌드하는 방법

## 참고
https://www.youtube.com/watch?v=E451K6Q0Ygw&list=PLJkjrxxiBSFAvGAVh6mYpGBLr_h0wqLnV  
https://www.youtube.com/watch?v=6A6cOvoydaM&list=PLJkjrxxiBSFAvGAVh6mYpGBLr_h0wqLnV&index=3  

---
# 20241230
# step설정, job설정, JPA 성능 문제와 JDBC

## step

우리는 이미 배치에서 Step을 만들었지만, 마지막으로 간단하게 Step에 대해서 알아보고 추가할 수 있는 안전 장치인 설정들에 대해 알아보겠습니다.  
Step은 배치 작업을 처리하는 하나의 묶음 입니다. 이런 Step은 두 가지 방식 중 하나로 구현됩니다.  

* Step
Chunk 방식 처리 (Read → Process → Write)
Tasklet 방식 처리

우리는 Chunk 단위 처리에 대해서만 알아보았습니다. Tasklet 방식은 아주 간단한 동작만 하기 때문에 데이터 조건을 걸어 이동하는데는 거의 사용하지 않습니다. (단순 파일 삭제, 값 초기화)  

따라서 Chunk 단위에 대해서 집중하면 될 거 같습니다. 아래서 Chunk 처리 과정에 대해서 간략하게 알아보겠습니다.  

## Step : Chunk 단위 처리 과정
청크 값을 10으로 설정 했다면  
(Read) X 10 → (Process) X 10 → Write  

## Skip
Skip은 Step의 과정 중 예외가 발생하게 되면 예외를 특정 수 까지 건너 뛸 수 있도록 설정하는 방법입니다.  

* 예시
```java
@Bean
public Step sixthStep() {

    return new StepBuilder("sixthStep", jobRepository)
            .<BeforeEntity, AfterEntity> chunk(10, platformTransactionManager)
            .reader(beforeSixthReader())
            .processor(middleSixthProcessor())
            .writer(afterSixthWriter())
            .faultTolerant()
            .skip(Exception.class)
            .noSkip(FileNotFoundException.class)
            .noSkip(IOException.class)
            .skipLimit(10)
            .build();
}
```
(skip과 noSkip의 순서는 무방함)  
`.skip(Exception.class)`이런식으로 전체 허용 해두고 
`.noSkip(FileNotFoundException.class)`특정한 부분 스킵 안되도록 많이설정  

* Skip을 조금 더 커스텀 하는 방법 (모든 예외를 허용)
```java
@Bean
public Step sixthStep() {

    return new StepBuilder("sixthStep", jobRepository)
            .<BeforeEntity, AfterEntity> chunk(10, platformTransactionManager)
            .reader(beforeSixthReader())
            .processor(middleSixthProcessor())
            .writer(afterSixthWriter())
            .faultTolerant()
            .skipPolicy(customSkipPolicy)
            .noSkip(FileNotFoundException.class)
            .noSkip(IOException.class)
            .build();
}
```

```java
@Configuration
public class CustomSkipPolicy implements SkipPolicy {

    @Override
    public boolean shouldSkip(Throwable t, long skipCount) throws SkipLimitExceededException {
        return true;
    }
}
```
커스텀 허용 함수 만들어주는 방법 있음  
https://docs.spring.io/spring-batch/reference/step/chunk-oriented-processing/configuring-skip.html  
커스텀 예외허용 공식문서  

## Retry
Retry는 Step의 과정 중 예외가 발생하게 되면 예외를 특정 수 까지 반복 할 수 있도록 설정하는 방법입니다.  

```java
@Bean
public Step sixthStep() {

    return new StepBuilder("sixthStep", jobRepository)
            .<BeforeEntity, AfterEntity> chunk(10, platformTransactionManager)
            .reader(beforeSixthReader())
            .processor(middleSixthProcessor())
            .writer(afterSixthWriter())
            .faultTolerant()
            .retryLimit(3)
            .retry(SQLException.class)
            .retry(IOException.class)
            .noRetry(FileNotFoundException.class)
            .build();
}
```
https://docs.spring.io/spring-batch/reference/step/chunk-oriented-processing/retry-logic.html  
Retry 공식 문서  

## Writer 롤백 제어
Writer시 특정 예외에 대해 트랜잭션 롤백을 제외하는 방법  

```java
@Bean
public Step step1(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
	return new StepBuilder("step1", jobRepository)
				.<String, String>chunk(2, transactionManager)
				.reader(itemReader())
				.writer(itemWriter())
				.faultTolerant()
				.noRollback(ValidationException.class)
				.build();
}
```
롤백 안하고 싶을때 설정가능  

## Step listener
stepListener는 Step의 실행 전후에 특정 작업을 수행 시킬 수 있는 방법입니다.  
로그를 남기거나 다음 Step이 준비가 되었는지, 이번 Step과 다음 Step이 의존되는 경우 변수 정리를 진행할 수 있습니다.  

```java
@Bean
public StepExecutionListener stepExecutionListener() {

    return new StepExecutionListener() {
        @Override
        public void beforeStep(StepExecution stepExecution) {
            StepExecutionListener.super.beforeStep(stepExecution);
        }

        @Override
        public ExitStatus afterStep(StepExecution stepExecution) {
            return StepExecutionListener.super.afterStep(stepExecution);
        }
    };
}

@Bean
public Step sixthStep() {

    return new StepBuilder("sixthStep", jobRepository)
            .<BeforeEntity, AfterEntity> chunk(10, platformTransactionManager)
            .reader(beforeSixthReader())
            .processor(middleSixthProcessor())
            .writer(afterSixthWriter())
            .listener(stepExecutionListener())
            .build();
}
```
Step실행 전후 뭔가 하고싶을때  

## Job 설정
실습 에서는 하나의 Step만 가지는 Job을 구성했지만 다양한 Step을 구성하고 조건을 둘 수 있는 방법에 대해 살펴보고,  
Job에 추가할 수 있는 여러 설정들을 알아보겠습니다.  

## Step flow

* 순차적으로 Step 실행
```java
@Bean
public Job footballJob(JobRepository jobRepository) {
    return new JobBuilder("footballJob", jobRepository)
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .build();
}
```
가장 첫 번째 실행될 Step만 start() 메소드로 설정한 뒤, next()로 이어주면 됩니다.  
다만 앞선 Step이 실패할 경우 연달아 등장하는 Step 또한 실행되지 않습니다.  

* 조건에 따라 실행
```java
@Bean
public Job job(JobRepository jobRepository, Step stepA, Step stepB, Step stepC, Step stepD) {
	return new JobBuilder("job", jobRepository)
				.start(stepA)
				.on("*").to(stepB)
				.from(stepA).on("FAILED").to(stepC)
				.from(stepA).on("COMPLETED").to(stepD)
				.end()
				.build();
}
```
성공했을때 다음 스텝 실패했을때 다음스탭 등 설정가능  
https://docs.spring.io/spring-batch/reference/step/controlling-flow.html  
flow공식문서  

## Job listener
jobListener는 Job의 실행 전후에 특정 작업을 수행 시킬 수 있는 방법입니다.  

```java
    @Bean
    public JobExecutionListener jobExecutionListener() {
        
        return new JobExecutionListener() {
            @Override
            public void beforeJob(JobExecution jobExecution) {
                JobExecutionListener.super.beforeJob(jobExecution);
            }

            @Override
            public void afterJob(JobExecution jobExecution) {
                JobExecutionListener.super.afterJob(jobExecution);
            }
        };
    }

    @Bean
    public Job sixthBatch() {

        return new JobBuilder("sixthBatch", jobRepository)
                .start(sixthStep())
                .listener(jobExecutionListener())
                .build();
    }
```

## JPA 성능 문제와 JDBC (JPA의 Write 성능 문제)
스프링 배치 read와 write 부분을 JPA로 구성할 경우 JDBC 대비 처리 속도가 엄청나게 저하됩니다.  
Reader의 경우 큰 영향을 미치진 않지만, Writer의 경우 엄청난 영향을 끼치는 이유는 아래와 같습니다.  

## 성능 저하 이유 : bulk 쿼리 실패
Entity의 Id 생성 전략은 보통 IDENTITY로 설정하게 됩니다. 이 설정은 `save() 수행시 DB 테이블을 조회하여 가장 마지막 값 보다 1을 증가 시킨 값을 저장하게 됩니다`.  
여기서 Batch 처리 청크 단위 bulk insert 수행이 무너지게 됩니다.  
JDBC 기반으로 작성하게 된다면 청크로 설정한 값이 모이게 된다면 bulk 쿼리로 단 1번의 insert가 수행되지만  
JPA의 IDENTITY 전략 때문에 bulk 쿼리 대신 각각의 수만큼 insert가 수행됩니다.  

## 비교 구현
* Reader
```java
@Bean
public RepositoryItemReader<BeforeEntity> beforeSixthReader() {

    return new RepositoryItemReaderBuilder<BeforeEntity>()
            .name("beforeReader")
            .pageSize(10)
            .methodName("findAll")
            .repository(beforeRepository)
            .sorts(Map.of("id", Sort.Direction.ASC))
            .build();
}

@Bean
public JdbcPagingItemReader<BeforeEntity> beforeSixthReader() {

    return new JdbcPagingItemReaderBuilder<BeforeEntity>()
            .name("beforeSixthReader")
            .dataSource(dataSource)
            .selectClause("SELECT id, username")
            .fromClause("FROM BeforeEntity")
            .sortKeys(Map.of("id", Order.ASCENDING))
            .rowMapper(new CustomBeforeRowMapper())
            .pageSize(10)
            .build();
}
```

* Writer
```java
@Bean
public RepositoryItemWriter<AfterEntity> afterSixthWriter() {

    return new RepositoryItemWriterBuilder<AfterEntity>()
            .repository(afterRepository)
            .methodName("save")
            .build();
}

@Bean
public JdbcBatchItemWriter<AfterEntity> afterSixthWriter() {

    String sql = "INSERT INTO AfterEntity (username) VALUES (:username)";

    return new JdbcBatchItemWriterBuilder<AfterEntity>()
            .dataSource(dataSource)
            .sql(sql)
            .itemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>())
            .build();
}
```

## 성능 측정
Job에 Listener 등록 후 시작 및 종료 시간 차이 측정  
순수 데이터 이동 104개  
2배정도 차이남  

이외에 상세하게 성능튜닝법은 네이버나 배민등 기술블로그에 잘나와있음  

## 참고
https://www.youtube.com/watch?v=IRTLwGOCpIw&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=13  
https://www.youtube.com/watch?v=FSjUJYZOtXo&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=15  
https://www.youtube.com/watch?v=GVSaAzYzCog&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=16  

---
# 20241227
# 배치 처리5 테이블 to 엑셀, ItemStreamReader

## 테이블 to 엑셀
다섯 번째 배치 작업은 데이터베이스 테이블에서 엑셀 파일 한 시트로 이동시키는 일 입니다.  

![image-337c1ba0-c9f6-491a-8ceb-adcceb75453c](https://github.com/user-attachments/assets/1fd0b2a3-b083-4761-9f6d-bce0d8fde5b9)  

"엑셀 → 테이블"과 “테이블 → 엑셀”의 차이를 생각하면 좋습니다.  
앞선 영상인 "엑셀 → 테이블"은 중간에 종료되더라도 중단점부터 실행하면 효율적입니다.  
하지만, 이번 작업은 “테이블 → 엑셀”이 실패하면 파일을 새로 만들어야되기 때문에 중단점이 아니라 처음부터 배치를 처리하도록 설정해야 합니다.  
(상황에 따라 다르겠지만, 파일을 만든다고 가정하면 새로 시작, 파일이 정의되어 있고 이어쓴다면 이어지는 부분 부터 진행하면 좋습니다.)  

## 엑셀 접근 의존성 추가
`implementation 'org.apache.poi:poi-ooxml:5.3.0'`  

## 테이블 : BeforeEntity

* Entity 정의
```java
@Entity
@Getter
@Setter
public class BeforeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
}
```

* Repository 정의
```java
public interface BeforeRepository extends JpaRepository<BeforeEntity, Long> {
}
```

## 테이블에 실습 데이터 넣기
* sql : insert into BeforeEntity
```java
INSERT INTO BeforeEntity (username) VALUES
('Alice Johnson'),
('Bob Smith'),
('Charlie Brown'),
('Diana Prince'),
('Edward Davis'),
('Fiona White'),
('George Harris'),
('Hannah Clark'),
('Ian Walker'),
('Jessica Lee'),
('Kevin Robinson'),
('Laura King'),
('Michael Scott'),
('Nina Evans'),
('Oscar Wright'),
('Pamela Adams'),
('Quincy Green'),
('Rachel Baker'),
('Sam Harris'),
('Tina Nelson'),
('Ursula Moore'),
('Victor Young'),
('Wendy Thompson'),
('Xander Garcia'),
('Yara Martinez'),
('Zachary Lewis'),
('Amelia Jones'),
('Benjamin Wilson'),
('Clara Taylor'),
('David Anderson'),
('Emma Martinez'),
('Frank Hernandez'),
('Grace Robinson'),
('Henry Walker'),
('Ivy Young'),
('Jack Scott'),
('Katherine Lee'),
('Liam Brown'),
('Mia Harris'),
('Noah King'),
('Olivia Adams'),
('Paul Nelson'),
('Quinn Green'),
('Riley White'),
('Sophia Davis'),
('Thomas Clark'),
('Uma Evans'),
('Vera Robinson'),
('William Wright'),
('Xena Baker'),
('Yusuf Harris'),
('Zoe Martinez'),
('Aaron Scott'),
('Bella Young'),
('Carlos Robinson'),
('Daisy Thompson'),
('Ethan Moore'),
('Faith Lewis'),
('Gina Clark'),
('Hank Green'),
('Iris Martinez'),
('James Taylor'),
('Kelly Adams'),
('Lucas King'),
('Maggie White'),
('Nathan Wilson'),
('Opal Harris'),
('Peter Scott'),
('Queenie Davis'),
('Ryan Baker'),
('Sandra Green'),
('Travis Johnson'),
('Ulysses Clark'),
('Vivian Martinez'),
('Walter White'),
('Xander Robinson'),
('Yvette King'),
('Zach Smith'),
('Allison Davis'),
('Bradley Moore'),
('Chloe Harris'),
('Daniel Green'),
('Emily Taylor'),
('Freddie Brown'),
('Gabriella Wilson'),
('Henry Scott'),
('Isabella White'),
('Jake Adams'),
('Kaitlyn Robinson'),
('Leo Clark'),
('Madison King'),
('Nina Brown'),
('Owen Martinez'),
('Peyton Harris'),
('Quinton Moore'),
('Rebecca White'),
('Steve Wilson'),
('Tara Scott'),
('Ursula Green'),
('Victor Harris'),
('Wendy Adams'),
('Xander King'),
('Yvonne Davis'),
('Zachary White');
```

## 클래스 생성 및 Job 정의
하나의 배치 Job을 정의할 클래스를 생성하고 Job 메소드를 등록해야 합니다.  
이때 배치 작업시 사용할 Repository 의존성들도 필드에 주입을 받도록 하겠습니다.  

```java
@Configuration
public class FifthBatch {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager platformTransactionManager;
    private final BeforeRepository beforeRepository;

    public FifthBatch(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager, BeforeRepository beforeRepository) {
        this.jobRepository = jobRepository;
        this.platformTransactionManager = platformTransactionManager;
        this.beforeRepository = beforeRepository;
    }

    @Bean
    public Job fifthJob() {

        System.out.println("fifth job");

        return new JobBuilder("fifthJob", jobRepository)
                .start(fifthStep())
                .build();
    }

    
}
```

## 순서 생각 및 Step 정의

Job을 정의 했지만, 실제 배치 처리는 Job 아래에 존재하는 하나의 Step에서 수행되게 됩니다.  
따라서 Step에서 “읽기 → 처리 → 쓰기” 과정을 구상해야 하며, Step을 등록하기 위한 @Bean을 등록하겠습니다.  

```java
@Bean
public Step fifthStep() {

    System.out.println("fifth step");

    return new StepBuilder("fifthStep", jobRepository)
            .<BeforeEntity, BeforeEntity> chunk(10, platformTransactionManager)
            .reader(fifthBeforeReader())
            .processor(fifthProcessor())
            .writer(excelWriter())
            .build();
}
```
정의한 Step은 Job의 start() 메소드 내부에 fourthStep()을 넣어주시면 됩니다.  

* 청크 : chunk
이때 읽기 → 처리 → 쓰기 작업은 청크 단위로 진행되는데, 대량의 데이터를 얼만큼 끊어서 처리할지에 대한 값으로 적당한 값을 선정해야 합니다.  
(너무 작으면 I/O 처리가 많아지고 오버헤드 발생, 너무 크면 적재 및 자원 사용에 대한 비용과 실패시 부담이 커짐)

## Read → Process → Write 작성
* Read : BeforeEntity 테이블에서 읽어오는 Reader
```java
@Bean
public RepositoryItemReader<BeforeEntity> fifthBeforeReader() {

    RepositoryItemReader<BeforeEntity> reader = new RepositoryItemReaderBuilder<BeforeEntity>()
            .name("beforeReader")
            .pageSize(10)
            .methodName("findAll")
            .repository(beforeRepository)
            .sorts(Map.of("id", Sort.Direction.ASC))
            .build();

    // 전체 데이터 셋에서 어디까지 수행 했는지의 값을 저장하지 않음
    reader.setSaveState(false);

    return reader;
}
```

* Process : 읽어온 데이터를 처리하는 Processor
```java
@Bean
public ItemProcessor<BeforeEntity, BeforeEntity> fifthProcessor() {

    return item -> item;
}
```

* Write : 엑셀 시트에 처리한 결과를 저장하는 Writer
```java
@Bean
public ItemStreamWriter<BeforeEntity> excelWriter() {

    try {
        return new ExcelRowWriter("C:\\Users\\kim\\Desktop\\result.xlsx");
        //리눅스나 맥은 /User/형태로
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```
우선은 ExcelWriter() 메소드를 만들고 실제 구현부는 따로 클래스를 빼도록 하겠습니다.  

```java
public class ExcelRowWriter implements ItemStreamWriter<BeforeEntity> {

    private final String filePath;
    private Workbook workbook;
    private Sheet sheet;
    private int currentRowNumber;
    private boolean isClosed;

    public ExcelRowWriter(String filePath) throws IOException {

        this.filePath = filePath;
        this.isClosed = false;
        this.currentRowNumber = 0;
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {
        workbook = new XSSFWorkbook();
        sheet = workbook.createSheet("Sheet1");
    }

    @Override
    public void write(Chunk<? extends BeforeEntity> chunk) {
        for (BeforeEntity entity : chunk) {
            Row row = sheet.createRow(currentRowNumber++);
            row.createCell(0).setCellValue(entity.getUsername());
        }
    }

    @Override
    public void close() throws ItemStreamException {

        if (isClosed) {
            return;
        }

        try (FileOutputStream fileOut = new FileOutputStream(filePath)) {
            workbook.write(fileOut);
        } catch (IOException e) {
            throw new ItemStreamException(e);
        } finally {
            try {
                workbook.close();
            } catch (IOException e) {
                throw new ItemStreamException(e);
            } finally {
                isClosed = true;
            }
        }
    }
}
```

## 손상된파일 오류날때

배치 어플리케이션을 종료하는 과정에서 이미 close()를 통해 정상적으로 닫힌 파일을 다시 핸들링하여 파일이 깨지는 이슈가 발생했습니다.  
위 문제를 해결하기 위해 파일의 상태를 체크하는 필드 변수를 만들었고 변경점에 대해 코드 변경을 완료하였습니다.  

## 전체코드

```java
@Configuration
public class FifthBatch {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager platformTransactionManager;
    private final BeforeRepository beforeRepository;

    public FifthBatch(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager, BeforeRepository beforeRepository) {
        this.jobRepository = jobRepository;
        this.platformTransactionManager = platformTransactionManager;
        this.beforeRepository = beforeRepository;
    }

    @Bean
    public Job fifthJob() {

        System.out.println("fifth job");

        return new JobBuilder("fifthJob", jobRepository)
                .start(fifthStep())
                .build();
    }

    @Bean
    public Step fifthStep() {

        System.out.println("fifth step");

        return new StepBuilder("fifthStep", jobRepository)
                .<BeforeEntity, BeforeEntity> chunk(10, platformTransactionManager)
                .reader(fifthBeforeReader())
                .processor(fifthProcessor())
                .writer(excelWriter())
                .build();
    }

    @Bean
    public RepositoryItemReader<BeforeEntity> fifthBeforeReader() {

        return new RepositoryItemReaderBuilder<BeforeEntity>()
                .name("beforeReader")
                .pageSize(10)
                .methodName("findAll")
                .repository(beforeRepository)
                .sorts(Map.of("id", Sort.Direction.ASC))
                .build();
    }

    @Bean
    public ItemProcessor<BeforeEntity, BeforeEntity> fifthProcessor() {

        return item -> item;
    }

    @Bean
    public ItemStreamWriter<BeforeEntity> excelWriter() {

        try {
            return new ExcelRowWriter("C:\\Users\\kim\\Desktop\\result.xlsx");
            //리눅스나 맥은 /User/형태로
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class ExcelRowWriter implements ItemStreamWriter<BeforeEntity> {

    private final String filePath;
    private Workbook workbook;
    private Sheet sheet;
    private int currentRowNumber;
    private boolean isClosed;

    public ExcelRowWriter(String filePath) throws IOException {

        this.filePath = filePath;
        this.isClosed = false;
        this.currentRowNumber = 0;
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {
        workbook = new XSSFWorkbook();
        sheet = workbook.createSheet("Sheet1");
    }

    @Override
    public void write(Chunk<? extends BeforeEntity> chunk) {
        for (BeforeEntity entity : chunk) {
            Row row = sheet.createRow(currentRowNumber++);
            row.createCell(0).setCellValue(entity.getUsername());
        }
    }

    @Override
    public void close() throws ItemStreamException {

        if (isClosed) {
            return;
        }

        try (FileOutputStream fileOut = new FileOutputStream(filePath)) {
            workbook.write(fileOut);
        } catch (IOException e) {
            throw new ItemStreamException(e);
        } finally {
            try {
                workbook.close();
            } catch (IOException e) {
                throw new ItemStreamException(e);
            } finally {
                isClosed = true;
            }
        }
    }
}
```

## ItemStreamReader 배치에서 데이터를 읽는 Reader

스프링 배치에서 가장 중요한 부분은 Reader 부분입니다. 현재까지 실행한 부분을 메타데이터에 저장해야하고 처리한 부분은 스킵해야 되기 때문입니다.  
스프링 배치에서 다양한 Reader 구현체를 제공합니다.  
하지만 내가 원하는 구현체가 없는 경우 직접 작성해야 하는데, 기본적인 Reader 인터페이스에 대한 Reader를 구현 방법에 대해서 간략하게 알아보겠습니다.  

## ItemStreamReader
앞선 엑셀 읽기 부분에서 사용했던 ItemStreamReader입니다.  
이 인터페이스는 배치의 초기화 및 상태를 관리할 수 있는 ItemStream과 실제 데이터 처리를 진행하는 ItemReader를 상속 받고 있습니다.  
ItemStreamReader = ItemStream + ItemReader  
```java
public interface ItemStreamReader<T> extends ItemStream, ItemReader<T> {

}
```

## ItemReader<>
```java
@FunctionalInterface
public interface ItemReader<T> {


	@Nullable
	T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;

}
```
* read()
배치 작업시 데이터를 읽기 위한 부분으로 하나의 데이터를 읽어올 때 read() 메소드가 호출됩니다.

## ItemStream
```java
public interface ItemStream {


	default void open(ExecutionContext executionContext) throws ItemStreamException {
	}


	default void update(ExecutionContext executionContext) throws ItemStreamException {
	}


	default void close() throws ItemStreamException {
	}

}
```
* open(ExecutionContext executionContext)
배치 처리가 시작되고 Step에서 처음 reader를 부르면 시작되는 부분으로 초기화나 이미 했던 작업의 경우 중단점 까지 건너 뛰도록 설계하는 부분입니다.

* update(ExecutionContext executionContext)
배치 작업시 read()와 함께 불려지는 메소드로 read() 호출 후 바로 호출되기 때문에 read()에서 처리한 작업 단위를 기록하는 용도로 사용됩니다.

* close()
배치 작업이 완료되고 불려지는 메소드로 파일을 저장하거나 필드 변수를 초기화하는 메소드로 사용됩니다.

## ExecutionContext
ItemStream의 open(), update()에 매개변수로 주입되는 있는 객체로 배치 작업 처리시 기준점을 잡을 변수를 계속하여 트래킹하기 위한 저장소로 사용됩니다.  
해당 클래스에서 put으로 값을 넣고, get으로 넣은 값을 가져옵니다.  

ExecutionContext 데이터는 JdbcExecutionContextDao에 의해 메타데이터 테이블에 저장되며 범위에 따라 아래와 같이 나뉩니다.  
* BATCH_JOB_EXECUTION_CONTEXT
* BATCH_STEP_EXECUTION_CONTEXT

## 구현 예시
실제로 동작하는 코드는 아니지만 예시로 작성 했습니다.  

아래 예시처럼 `open`함수처럼 현재의 상태를 트레킹하는부분이 꼭필요 대부분 `read`함수만 만들어쓰고 마는 경우가 많음  

```java
public class CustomItemStreamReaderImpl implements ItemStreamReader<String> {

    private final RestTemplate restTemplate;
    private int currentId;
    private final String CURRENT_ID_KEY = "current.call.id";
    private final String API_URL = "https://www.devyummi.com/page?id=";

    public CustomItemStreamReaderImpl(RestTemplate restTemplate) {

        this.currentId = 0;
        this.restTemplate = restTemplate;
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {

        if (executionContext.containsKey(CURRENT_ID_KEY)) {
            currentId = executionContext.getInt(CURRENT_ID_KEY);
        }
    }

    @Override
    public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {

        currentId++;

        String url = API_URL + currentId;
        String response = restTemplate.getForObject(url, String.class);

        if (response == null) {
            return null;
        }
        return response;
    }

    @Override
    public void update(ExecutionContext executionContext) throws ItemStreamException {
        executionContext.putInt(CURRENT_ID_KEY, currentId);
    }

    @Override
    public void close() throws ItemStreamException {

    }
}
```

## 참조
https://www.youtube.com/watch?v=G_yVcism96A&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=11  
https://www.youtube.com/watch?v=4D65kFIPUyo&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=13  


---
# 20241016
# 배치처리1 실행, 스케쥴, 배치처리2 테이블조건

## 배치1 실행
저번 시간에 작성한 테이블 → 테이블 데이터 이동 배치를 실행시켜야 합니다.  
우리는 application.properties에서 배치 자동 실행에 대한 변수 값을 false로 설정 했기 때문에 배치를 실행시키기 위한 jobLauncher를 구현해야 합니다.  

## jobLauncher 및 실행 변수
정의한 Job을 실행 시키기 위해서는 아래와 같은 구현을 진행해야 합니다.  
(예시 로직, 실제로 동작하지 않는 슈도 코드)  
```java
private final JobLauncher jobLauncher;
private final JobRegistry jobRegistry;

JobParameters jobParameters = new JobParametersBuilder()
        .addString("date", value)
        .toJobParameters();

jobLauncher.run(jobRegistry.getJob("firstJob"), jobParameters);
```
필요한 모듈을 사용하면서 파라미터를 통해서 중복실행을 방지합니다.  

## 컨트롤러에서 실행
“/first” 경로로 요청이 들어온다면, 배치 1을 실행하도록 설정하겠습니다.  
이때 배치 실행에 대한 판단 값도 함께 넘겨줘야 중복 실행 및 실행 스케줄을 확인할 수 있습니다.  

```java
@Controller
@ResponseBody
public class MainController {

    private final JobLauncher jobLauncher;
    private final JobRegistry jobRegistry;

    public MainController(JobLauncher jobLauncher, JobRegistry jobRegistry) {
        this.jobLauncher = jobLauncher;
        this.jobRegistry = jobRegistry;
    }

    @GetMapping("/first")
    public String firstApi(@RequestParam("value") String value) throws Exception {

        JobParameters jobParameters = new JobParametersBuilder()
                .addString("date", value)
                .toJobParameters();

        jobLauncher.run(jobRegistry.getJob("firstJob"), jobParameters);


        return "ok";
    }
}
```
동기적으로 처리되기 때문에 요청 → 처리 → 응답에 대한 딜레이가 발생한다. (callable과 같은 도구로 비동기 처리를 진행해도 좋다.)  

## 스케쥴로 실행
* Main 클래스에 스케쥴 활성화 어노테이션 등록
```java
@SpringBootApplication
@EnableScheduling
public class SpringBatchApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBatchApplication.class, args);
    }
}
```

* Schedule Config  클래스 생성
```java
@Configuration
public class FirstSchedule {

    private final JobLauncher jobLauncher;
    private final JobRegistry jobRegistry;

    public FirstSchedule(JobLauncher jobLauncher, JobRegistry jobRegistry) {
        this.jobLauncher = jobLauncher;
        this.jobRegistry = jobRegistry;
    }

    @Scheduled(cron = "10 * * * * *", zone = "Asia/Seoul")
    public void runFirstJob() throws Exception {

        System.out.println("first schedule start");

        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd-hh-mm-ss");
        String date = dateFormat.format(new Date());

        JobParameters jobParameters = new JobParametersBuilder()
                .addString("date", date)
                .toJobParameters();

        jobLauncher.run(jobRegistry.getJob("firstJob"), jobParameters);
    }
}
```

## JobLauncher에서 JobParameter
JobLauncher로 Job 실행시 JobParameter를 주는 이유는 실행한 작업에 대한 일자, 순번등을 부여해 동일한 일자에 대한 작업의 수행 여부를 확인하여 중복 실행 및 미실행을 예방할 수 있습니다.  

만약 `/first/?value=a` 로 실행시킨후 다시 `/first/?value=a`로 요청을 보내면 실행되지 않습니다.  
`/first/?value=b`로 바꾸게 되면 다시 실행이 됩니다.  

## 배치 처리 2 : 테이블 조건 확인 후 값 변경
두 번째 배치 작업은 하나의 테이블에서 조건을 통해 데이터를 읽은 후 데이터를 변경하고 다시 테이블에 저장하는 일 입니다.  
이번 작업은 테이블에서 데이터를 읽어 “win” 컬럼 값이 10이 넘으면 “reward” 컬럼에 true 값을 주는 것 입니다.  

![1](https://github.com/user-attachments/assets/540a7256-cba4-4753-bb0f-149c49cc4bfc)  

## 테이블 : WinEntity

* Entity 정의
```java
@Entity
@Getter
@Setter
public class WinEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private Long win;
    private Boolean reward;
}
```

* Repository 정의
```java
public interface WinRepository extends JpaRepository<WinEntity, Long> {

    Page<WinEntity> findByWinGreaterThanEqual(Long win, Pageable pageable);
}
```

## 테이블에 실습 데이터 넣기
* sql : insert into WinEntity
```sql
INSERT INTO WinEntity (id, reward, username, win) VALUES
(1, FALSE, 'xxxjjhhh', 10),
(2, FALSE, 'devyummi', 1),
(3, FALSE, 'jihun', 3),
(4, FALSE, 'user4', 20),
(5, FALSE, 'user5', 20),
(6, FALSE, 'user6', 7),
(7, FALSE, 'user7', 13),
(8, FALSE, 'user8', 11),
(9, FALSE, 'user9', 14),
(10, FALSE, 'user10', 0),
(11, FALSE, 'user11', 5),
(12, FALSE, 'user12', 6),
(13, FALSE, 'user13', 13),
(14, FALSE, 'user14', 17),
(15, FALSE, 'user15', 8),
(16, FALSE, 'user16', 3),
(17, FALSE, 'user17', 9),
(18, FALSE, 'user18', 11),
(19, FALSE, 'user19', 13),
(20, FALSE, 'user20', 19),
(21, FALSE, 'user21', 5),
(22, FALSE, 'user22', 3),
(23, FALSE, 'user23', 1),
(24, FALSE, 'user24', 0),
(25, FALSE, 'user25', 0),
(26, FALSE, 'user26', 1),
(27, FALSE, 'user27', 3),
(28, FALSE, 'user28', 4),
(29, FALSE, 'user29', 12),
(30, FALSE, 'user30', 11),
(31, FALSE, 'user31', 10),
(32, FALSE, 'user32', 20),
(33, FALSE, 'user33', 2),
(34, FALSE, 'user34', 5),
(35, FALSE, 'user35', 1),
(36, FALSE, 'user36', 16),
(37, FALSE, 'user37', 7),
(38, FALSE, 'user38', 6),
(39, FALSE, 'user39', 10),
(40, FALSE, 'user40', 20),
(41, FALSE, 'user41', 19),
(42, FALSE, 'user42', 2),
(43, FALSE, 'user43', 3),
(44, FALSE, 'user44', 4),
(45, FALSE, 'user45', 4),
(46, FALSE, 'user46', 14),
(47, FALSE, 'user47', 2),
(48, FALSE, 'user48', 0),
(49, FALSE, 'user49', 19),
(50, FALSE, 'user50', 13),
(51, FALSE, 'user51', 0),
(52, FALSE, 'user52', 12),
(53, FALSE, 'user53', 19),
(54, FALSE, 'user54', 7),
(55, FALSE, 'user55', 9),
(56, FALSE, 'user56', 7),
(57, FALSE, 'user57', 2),
(58, FALSE, 'user58', 20),
(59, FALSE, 'user59', 19),
(60, FALSE, 'user60', 18),
(61, FALSE, 'user61', 12),
(62, FALSE, 'user62', 11),
(63, FALSE, 'user63', 1),
(64, FALSE, 'user64', 0),
(65, FALSE, 'user65', 21),
(66, FALSE, 'user66', 4),
(67, FALSE, 'user67', 5),
(68, FALSE, 'user68', 8),
(69, FALSE, 'user69', 2),
(70, FALSE, 'user70', 3),
(71, FALSE, 'user71', 11),
(72, FALSE, 'user72', 11),
(73, FALSE, 'user73', 11),
(74, FALSE, 'user74', 7),
(75, FALSE, 'user75', 18),
(76, FALSE, 'user76', 19),
(77, FALSE, 'user77', 12),
(78, FALSE, 'user78', 3),
(79, FALSE, 'user79', 7),
(80, FALSE, 'user80', 7),
(81, FALSE, 'user81', 8),
(82, FALSE, 'user82', 5),
(83, FALSE, 'user83', 5),
(84, FALSE, 'user84', 2),
(85, FALSE, 'user85', 1),
(86, FALSE, 'user86', 3),
(87, FALSE, 'user87', 8),
(88, FALSE, 'user88', 9),
(89, FALSE, 'user89', 12),
(90, FALSE, 'user90', 11),
(91, FALSE, 'user91', 1),
(92, FALSE, 'user92', 3),
(93, FALSE, 'user93', 2),
(94, FALSE, 'user94', 19),
(95, FALSE, 'user95', 20),
(96, FALSE, 'user96', 23),
(97, FALSE, 'user97', 1),
(98, FALSE, 'user98', 0),
(99, FALSE, 'user99', 3),
(100, FALSE, 'batchhh100', 8);
```

## 스프링 배치 모식도
![1](https://github.com/user-attachments/assets/097608b9-32bf-4784-b64b-7e0a7eca8c3c)  

## 클래스 생성 및 Job 정의
기본적으로 하나의 배치 Job을 정의할 클래스를 생성하고 Job 메소드를 등록해야 합니다.  
이때 배치 작업시 사용할 Repository 의존성들도 필드에 주입을 받도록 하겠습니다.  

```java
@Configuration
public class SecondBatch {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager platformTransactionManager;
    private final WinRepository winRepository;

    public SecondBatch(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager, WinRepository winRepository) {
        this.jobRepository = jobRepository;
        this.platformTransactionManager = platformTransactionManager;
        this.winRepository = winRepository;
    }

    @Bean
    public Job secondJob() {

        return new JobBuilder("secondJob", jobRepository)
                .start(스텝들어갈자리)
                .build();
    }
}
```

## 순서 생각 및 Step 정의
Job을 정의 했지만, 실제 배치 처리는 Job 아래에 존재하는 하나의 Step에서 수행되게 됩니다.  
따라서 Step에서 “읽기 → 처리 → 쓰기” 과정을 구성해야 하며, Step을 등록하기 위한 @Bean을 등록하겠습니다.  

```java
@Bean
public Step secondStep() {

    return new StepBuilder("secondStep", jobRepository)
            .<WinEntity, WinEntity> chunk(10, platformTransactionManager)
            .reader(winReader())
            .processor(trueProcessor())
            .writer(winWriter())
            .build();
}
```
정의한 Step은 Job의 start() 메소드 내부에 secondStep()을 넣어주시면 됩니다.  

* 청크 : chunk
이때 읽기 → 처리 → 쓰기 작업은 청크 단위로 진행되는데, 대량의 데이터를 얼만큼 끊어서 처리할지에 대한 값으로 적당한 값을 선정해야 합니다.  
(너무 작으면 I/O 처리가 많아지고 오버헤드 발생, 너무 크면 적재 및 자원 사용에 대한 비용과 실패시 부담이 커짐)

## Read → Process → Write 작성
* Read : WinEntity 테이블에서 읽어오는 Reader
```java
@Bean
public RepositoryItemReader<WinEntity> winReader() {

    return new RepositoryItemReaderBuilder<WinEntity>()
            .name("winReader")
            .pageSize(10)
            .methodName("findByWinGreaterThanEqual")
            .arguments(Collections.singletonList(10L))
            .repository(winRepository)
            .sorts(Map.of("id", Sort.Direction.ASC))
            .build();
}
```
아주 다양한 Reader 인터페이스와 구현체들이 존재하지만, 우리는 JPA를 통한 쿼리를 수행하기 때문에 RepositoryItemReader를 사용합니다.  
이때 데이터는 페이징을 통해 읽기 때문에 Repository에 존재하는 JPA 메소드도 Page<> 타입을 반환하도록 작성해야 합니다.  

* Process : 읽어온 데이터를 처리하는 Process (큰 작업을 수행하지 않을 경우 생략 가능, 지금과 같이 단순 컬럼값 변경은 필요 없음)
```java
@Bean
public ItemProcessor<WinEntity, WinEntity> trueProcessor() {

    return item -> {
        item.setReward(true);
        return item;
    };
}
```

* Write : WinEntity에 처리한 결과를 저장
```java
@Bean
public RepositoryItemWriter<WinEntity> winWriter() {

    return new RepositoryItemWriterBuilder<WinEntity>()
            .repository(winRepository)
            .methodName("save")
            .build();
}
```

## 전체 코드
```java
@Configuration
public class SecondBatch {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager platformTransactionManager;
    private final WinRepository winRepository;

    public SecondBatch(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager, WinRepository winRepository) {
        this.jobRepository = jobRepository;
        this.platformTransactionManager = platformTransactionManager;
        this.winRepository = winRepository;
    }

    @Bean
    public Job secondJob() {

        return new JobBuilder("secondJob", jobRepository)
                .start(secondStep())
                .build();
    }

    @Bean
    public Step secondStep() {

        return new StepBuilder("secondStep", jobRepository)
                .<WinEntity, WinEntity> chunk(10, platformTransactionManager)
                .reader(winReader())
                .processor(trueProcessor())
                .writer(winWriter())
                .build();
    }

    @Bean
    public RepositoryItemReader<WinEntity> winReader() {

        return new RepositoryItemReaderBuilder<WinEntity>()
                .name("winReader")
                .pageSize(10)
                .methodName("findByWinGreaterThanEqual")
                .arguments(Collections.singletonList(10L))
                .repository(winRepository)
                .sorts(Map.of("id", Sort.Direction.ASC))
                .build();
    }

    @Bean
    public ItemProcessor<WinEntity, WinEntity> trueProcessor() {

        return item -> {
            item.setReward(true);
            return item;
        };
    }

    @Bean
    public RepositoryItemWriter<WinEntity> winWriter() {

        return new RepositoryItemWriterBuilder<WinEntity>()
                .repository(winRepository)
                .methodName("save")
                .build();
    }
}
```

이 배치도 이전시간에 배운 스케쥴이나 엔드포인트를 통해서 시작점을 만들어주면 사용할 수 있습니다.  

## 참고
https://www.youtube.com/watch?v=ltKfPkD2wvE&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=7  
https://www.youtube.com/watch?v=X1lGjIeNHF8&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=9  

---
# 20241015
# 스프링배치 메타데이터 테이블, 처리1 테이블 TO 테이블

## 메타데이터 테이블이란?
배치에서 중요한 작업에 대한 트래킹을 수행하는 테이블로, 스프링 배치에서도 메타데이터를 관리해야 합니다.  
보통은 DB 테이블에 저장하며, application.properties 변수 설정시 @Primary로 설정한 테이블에 테이블을 자동으로 생성할 수 있습니다.  

## 메타데이터 테이블 생성 설정

* application.properties
```
spring.batch.jdbc.initialize-schema=always
spring.batch.jdbc.schema=classpath:org/springframework/batch/core/schema-mysql.sql
```
always를 설정하면 batch 설정에서 jdbc를 통해 테이블을 생성하며 생성 쿼리는 연결된 DB를 파악한 뒤 자동으로 찾지만, 혹시 찾지 못하거나 커스텀을 원하는 경우 classpath값을 부여할 수 있습니다.  

## 테이블 ERD

![1](https://github.com/user-attachments/assets/08b4942f-e2c0-40a1-8eba-cc887b24c655)  

메타데이터 저장에 필요한 내용들을 사용 DB에 맞게 생성해줍니다. 상세한 내용은 공식문서 참고  
https://docs.spring.io/spring-batch/reference/schema-appendix.html  

## 테이블 스크립트 경로 확인하기
External Libararies > Gradle: org.springframework.batch:spring-batch-core:버전 > org.springframework.batch.core  

![1](https://github.com/user-attachments/assets/3e1a5ae1-fafa-4837-844a-0acebb3c9099)  

## DB에서 생성된 테이블 확인

![1](https://github.com/user-attachments/assets/44b2a9e5-072a-485c-ab3f-9fefbecc8a5b)  

## 배치 처리1 : 테이블 to 테이블 기초 테이블 간 데이터 이동

우리의 첫 번째 배치 작업은 간단하게 2개의 테이블을 정의하고 하나의 테이블에 있는 데이터 전부를 다른 테이블로 이동 시키는 일 입니다.  
실제 프로덕션에서는 이렇게 사용할 일이 거의 없겠지만 배치를 이해하기위해 만들어 봅니다.  

![1](https://github.com/user-attachments/assets/931a7a07-716f-49f6-89a7-dcafdf1a898b)  

## 테이블1 : BeforeEntity

* Entity 정의
```java
@Entity
@Getter
@Setter
public class BeforeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
}
```

* Repository 정의
```java
public interface BeforeRepository extends JpaRepository<BeforeEntity, Long> {

}
```

## 테이블2 : AfterEntity

* Entity 정의
```java
@Entity
@Getter
@Setter
public class AfterEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
}
```

* Repository 정의
```java
public interface AfterRepository extends JpaRepository<AfterEntity, Long> {

}
```

## 테이블1에 실습 데이터 넣기

* sql : insert into BeforeEntity
```sql
INSERT INTO BeforeEntity (username) VALUES
('Alice Johnson'),
('Bob Smith'),
('Charlie Brown'),
('Diana Prince'),
('Edward Davis'),
('Fiona White'),
('George Harris'),
('Hannah Clark'),
('Ian Walker'),
('Jessica Lee'),
('Kevin Robinson'),
('Laura King'),
('Michael Scott'),
('Nina Evans'),
('Oscar Wright'),
('Pamela Adams'),
('Quincy Green'),
('Rachel Baker'),
('Sam Harris'),
('Tina Nelson'),
('Ursula Moore'),
('Victor Young'),
('Wendy Thompson'),
('Xander Garcia'),
('Yara Martinez'),
('Zachary Lewis'),
('Amelia Jones'),
('Benjamin Wilson'),
('Clara Taylor'),
('David Anderson'),
('Emma Martinez'),
('Frank Hernandez'),
('Grace Robinson'),
('Henry Walker'),
('Ivy Young'),
('Jack Scott'),
('Katherine Lee'),
('Liam Brown'),
('Mia Harris'),
('Noah King'),
('Olivia Adams'),
('Paul Nelson'),
('Quinn Green'),
('Riley White'),
('Sophia Davis'),
('Thomas Clark'),
('Uma Evans'),
('Vera Robinson'),
('William Wright'),
('Xena Baker'),
('Yusuf Harris'),
('Zoe Martinez'),
('Aaron Scott'),
('Bella Young'),
('Carlos Robinson'),
('Daisy Thompson'),
('Ethan Moore'),
('Faith Lewis'),
('Gina Clark'),
('Hank Green'),
('Iris Martinez'),
('James Taylor'),
('Kelly Adams'),
('Lucas King'),
('Maggie White'),
('Nathan Wilson'),
('Opal Harris'),
('Peter Scott'),
('Queenie Davis'),
('Ryan Baker'),
('Sandra Green'),
('Travis Johnson'),
('Ulysses Clark'),
('Vivian Martinez'),
('Walter White'),
('Xander Robinson'),
('Yvette King'),
('Zach Smith'),
('Allison Davis'),
('Bradley Moore'),
('Chloe Harris'),
('Daniel Green'),
('Emily Taylor'),
('Freddie Brown'),
('Gabriella Wilson'),
('Henry Scott'),
('Isabella White'),
('Jake Adams'),
('Kaitlyn Robinson'),
('Leo Clark'),
('Madison King'),
('Nina Brown'),
('Owen Martinez'),
('Peyton Harris'),
('Quinton Moore'),
('Rebecca White'),
('Steve Wilson'),
('Tara Scott'),
('Ursula Green'),
('Victor Harris'),
('Wendy Adams'),
('Xander King'),
('Yvonne Davis'),
('Zachary White');
```
테스트해볼 더미 데이터로 DB에 넣어주고 테스트 해봅니다.  

## 스프링 배치 모식도
![1](https://github.com/user-attachments/assets/097608b9-32bf-4784-b64b-7e0a7eca8c3c)  

## 클래스 생성 및 Job 정의
기본적으로 하나의 배치 Job을 정의할 클래스를 생성하고 Job 메소드를 등록해야 합니다.  
이때 배치 작업시 사용할 Repository 의존성들도 필드에 주입을 받도록 하겠습니다.  

```java
 @Configuration
public class FirstBatch {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager platformTransactionManager;

    private final BeforeRepository beforeRepository;
    private final AfterRepository afterRepository;

    public FirstBatch(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager, BeforeRepository beforeRepository, AfterRepository afterRepository) {

        this.jobRepository = jobRepository;
        this.platformTransactionManager = platformTransactionManager;
        this.beforeRepository = beforeRepository;
        this.afterRepository = afterRepository;
    }

    @Bean
    public Job firstJob() {

        System.out.println("first job");

        return new JobBuilder("firstJob", jobRepository)
                .start(스텝들어갈자리) //다음꺼 필요하면 .next()이런식으로 추가해서 넣어줄 수 있음
                .build();
    }
}
```

## 순서 생각 및 Step 정의
Job을 정의 했지만, 실제 배치 처리는 Job 아래에 존재하는 하나의 Step에서 수행되게 됩니다.  
따라서 Step에서 "읽기 → 처리 → 쓰기" 과정을 구상해야 하며, Step을 등록하기 위한 @Bean을 등록하겠습니다.  

```java
@Bean
public Step firstStep() {

    System.out.println("first step");

    return new StepBuilder("firstStep", jobRepository)
            .<BeforeEntity, AfterEntity> chunk(10, platformTransactionManager)
            .reader(읽는메소드자리)
            .processor(처리메소드자리)
            .writer(쓰기메소드자리)
            .build();
}
```

정의한 Step은 Job의 start() 메소드 내부에 firstStep()을 넣어주시면 됩니다.  

* 청크 : chunk
이때 읽기 → 처리 → 쓰기 작업은 청크 단위로 진행되는데, 대량의 데이터를 얼만큼 끊어서 처리할지에 대한 값으로 적당한 값을 선정해야 합니다.  
(너무 작으면 I/O 처리가 많아지고 오버헤드 발생, 너무 크면 적재 및 자원 사용에 대한 비용과 실패시 부담이 커짐)

## Read → Process → Write 작성

* Read : BeforeEntity 테이블에서 읽어오는 Reader
```java
@Bean
public RepositoryItemReader<BeforeEntity> beforeReader() {

    return new RepositoryItemReaderBuilder<BeforeEntity>() //사용 엔티티정의
            .name("beforeReader") //이름
            .pageSize(10) //10개씩만 가져오겠다
            .methodName("findAll") //findAll 쿼리 사용할것
            .repository(beforeRepository) 
            .sorts(Map.of("id", Sort.Direction.ASC)) // id로 sort해서 가져온다.
            .build();
}
```
아주 다양한 Reader 인터페이스와 구현체들이 존재하지만, 우리는 JPA를 통한 쿼리를 수행하기 때문에 RepositoryItemReader를 사용합니다.  
이때 청크 단위까지만 읽기 때문에 findAll을 하더라도 chunk 개수 만큼 사용하게 됩니다.  
따라서 자원 낭비를 방지하기 위해 Sort를 진행하고 pageSize() 단위를 설정해 findAll이 아닌 페이지 만큼 읽어올 수 있도록 설정합니다.  

* Process : 읽어온 데이터를 처리하는 Process (큰 작업을 수행하지 않을 경우 생략 가능, 지금과 같이 단순 이동은 사실 필요 없음)
```java
@Bean
public ItemProcessor<BeforeEntity, AfterEntity> middleProcessor() {

    return new ItemProcessor<BeforeEntity, AfterEntity>() {

        @Override
        public AfterEntity process(BeforeEntity item) throws Exception { //이전 Read에서 가져온 객체 item이름으로 가져옴

            AfterEntity afterEntity = new AfterEntity();
            afterEntity.setUsername(item.getUsername()); 

            return afterEntity;
        }
    };
}
```
(람다식으로 변경 가능)  

* Write : AfterEntity에 처리한 결과를 저장하는 Writer
```java
@Bean
public RepositoryItemWriter<AfterEntity> afterWriter() {

    return new RepositoryItemWriterBuilder<AfterEntity>() //엔티티로 save 쿼리 저장
            .repository(afterRepository)
            .methodName("save")
            .build();
}
```
Writer 또한 다양한 인터페이스 및 구현체가 존재하지만 JPA를 통한 쿼리를 날리는게 목표로 RepositoryItemWriter를 사용합니다.  

이렇게 일차적으로 완성된 코드는 바로 동작은 안합니다. 우리가 원하는 트리거로 원하는 때에 동작시키고 싶어 properties에 job.enalbe을 false로 해두었기때문입니다.   
다음 학습에서 이를 세팅해보겠습니다.  

## 전체 코드
```java
@Configuration
public class FirstBatch {

    private final JobRepository jobRepository;
    private final PlatformTransactionManager platformTransactionManager;

    private final BeforeRepository beforeRepository;
    private final AfterRepository afterRepository;

    public FirstBatch(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager, BeforeRepository beforeRepository, AfterRepository afterRepository) {

        this.jobRepository = jobRepository;
        this.platformTransactionManager = platformTransactionManager;
        this.beforeRepository = beforeRepository;
        this.afterRepository = afterRepository;
    }

    @Bean
    public Job firstJob() {

        System.out.println("first job");

        return new JobBuilder("firstJob", jobRepository)
                .start(firstStep())
                .build();
    }

    @Bean
    public Step firstStep() {

        System.out.println("first step");

        return new StepBuilder("firstStep", jobRepository)
                .<BeforeEntity, AfterEntity> chunk(10, platformTransactionManager)
                .reader(beforeReader())
                .processor(middleProcessor())
                .writer(afterWriter())
                .build();
    }

    @Bean
    public RepositoryItemReader<BeforeEntity> beforeReader() {

        return new RepositoryItemReaderBuilder<BeforeEntity>()
                .name("beforeReader")
                .pageSize(10)
                .methodName("findAll")
                .repository(beforeRepository)
                .sorts(Map.of("id", Sort.Direction.ASC))
                .build();
    }

    @Bean
    public ItemProcessor<BeforeEntity, AfterEntity> middleProcessor() {

        return new ItemProcessor<BeforeEntity, AfterEntity>() {

            @Override
            public AfterEntity process(BeforeEntity item) throws Exception {

                AfterEntity afterEntity = new AfterEntity();
                afterEntity.setUsername(item.getUsername());

                return afterEntity;
            }
        };
    }

    @Bean
    public RepositoryItemWriter<AfterEntity> afterWriter() {

        return new RepositoryItemWriterBuilder<AfterEntity>()
                .repository(afterRepository)
                .methodName("save")
                .build();
    }
}
```

## 참고
https://www.youtube.com/watch?v=1ABbCRZh5v4&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=5  
https://www.youtube.com/watch?v=vY_UGvWTh0Q&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=7 

---
# 20241011
# 스프링배치 프로젝트 생성, DB 연결

## 의존성
* Lombok
* Spring Web
* JDBC API
* Spring Data JPA
* MySQL Driver
* Spring Batch

* build.gradle
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-batch'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.mysql:mysql-connector-j'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.batch:spring-batch-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

## 배치 자동실행방지
스프링 부트 및 스프링 배치는 1개의 배치 작업에 대해 프로젝트를 실행하면 자동으로 배치 작업이 가동되기 때문에 해당 과정을 막아야 한다.  

* application.properties
`spring.batch.job.enabled=false`

## 스프링부트프로젝트 패키지 생성
* batch (배치학습중이라 밖으로 뺌 원래는 config안으로)
* config
* controller
* entity
* repository
* schedule

## 2개의 데이터베이스를 연결
2개의 데이터베이스를 통해 시리즈 진행 (단일 연결은 단순 변수 셋팅으로 정말 쉽지만, 자주 사용하는 경우가 없을거 같습니다.)  
배치를 사용하면 대부분 두개의 데이터베이스를 사용하게됨  

![1](https://github.com/user-attachments/assets/6d7a9b8f-5762-4be5-93e1-2522055efa89)  

* 1 : 메타데이터용
배치 작업의 진행 사항 및 내용에 대한 메타데이터를 기록하는 테이블을 위한 DB  

* 2 : 데이터소스용
배치 작업 데이터용 DB

## 데이터베이스 연결 변수 작성
* application.properties
```
spring.datasource-meta.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource-meta.jdbc-url=jdbc:mysql://아이피:3306/첫번째디비명?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul&allowPublicKeyRetrieval=true
spring.datasource-meta.username=아이디
spring.datasource-meta.password=비밀번호

spring.datasource-data.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource-data.jdbc-url=jdbc:mysql://아이피:3306/두번째디비명?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul&allowPublicKeyRetrieval=true
spring.datasource-data.username=아이디
spring.datasource-data.password=비밀번호
```

스프링 부트는 하나의 데이터베이스에 대해서만 변수 방식으로 자동 연결을 진행하기 때문에 2개 등록시 등록을 진행하는 Config 클래스를 작성해야 합니다.  
따라서 위 변수 값들을 기반으로 다음 영상에서 DB 연결 Config 클래스를 작성하도록 하겠습니다.  
지금상태로 서버를 시작하면 오류가 나게됩니다.  

스프링 부트에서 2개 이상의 DB를 연결하려면 Config 클래스를 필수적으로 작성해야 하며, 충돌을 방지하여 우선 순위를 위해 @Primary Config를 설정해야 합니다.  
이때 스프링 배치의 기본적인 메타데이터는 @Primary로 잡혀 있는 DB 소스에 초기화되게 됩니다.  

## MetaDBConfig : 첫 번째 DB
* config > MetaDBConfig.java
```java
@Configuration
public class MetaDBConfig {

    @Primary
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource-meta")
    public DataSource metaDBSource() {

        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean
    public PlatformTransactionManager metaTransactionManager() {

        return new DataSourceTransactionManager(metaDBSource());
    }
}
```
`@Primary`로 설정할 내용들을 `prefix = "spring.datasource-meta"`로 가져와서 등록해줌

## DataDBConfig : 두 번째 DB
* config > DataDBConfig.java
```java
@Configuration
@EnableJpaRepositories(
        basePackages = "com.example.samplebatch.repository",
        entityManagerFactoryRef = "dataEntityManager",
        transactionManagerRef = "dataTransactionManager"
)
public class DataDBConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource-data")
    public DataSource dataDBSource() {

        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean dataEntityManager() {

        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();

        em.setDataSource(dataDBSource());
        em.setPackagesToScan(new String[]{"com.example.samplebatch.entity"});
        em. setJpaVendorAdapter(new HibernateJpaVendorAdapter());

        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto", "update");
        properties.put("hibernate.show_sql", "true");
        em.setJpaPropertyMap(properties);

        return em;
    }

    @Bean
    public PlatformTransactionManager dataTransactionManager() {

        JpaTransactionManager transactionManager = new JpaTransactionManager();

        transactionManager.setEntityManagerFactory(dataEntityManager().getObject());

        return transactionManager;
    }
}
```
일반적으로 사용할 DB 내용들을 어떤 엔티티에 적용할지 `@EnableJpaRepositories`에 세부적으로 적어주고 `em`에 설정된 값을 등록  
`properties.put("hibernate.hbm2ddl.auto", "update");`같은건 이렇게 2개의 DB를 사용할때는 properties 파일에 적어서 등록할수 없어서 따로 함수로 넣어줌  

이렇게 셋팅후에는 정상적으로 서버가 기동가능하다.  

## 참조
https://www.youtube.com/watch?v=5jYhq28Vri4&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=3  
https://www.youtube.com/watch?v=OfMM4BbBlpY&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=5  

---
# 20241010
# 스프링배치 학습목표와 동작원리  

## 목표
스프링 배치 5 프레임워크를 활용하여 스프링 생태계에서 대량의 데이터를 안전하게 처리할 수 있는 기본적인 환경을 구축한다.

## 배치란? : 아주 간단하게
우선 사전적 의미는 “일정 시간 동안 대량의 데이터를 한 번에 처리하는 방식”을 의미합니다.  
이때 프레임워크를 사용하는 이유는? 아주 많은 데이터를 처리 하는 중간에 프로그램이 멈출 수 있는 상황을 대비해 안전 장치를 마련해야 하기 때문입니다.  
10만개의 데이터를 복잡한 JOIN을 걸어 DB간 이동 시키는 도중 프로그램이 멈춰버리면 처음부터 다시 시작할 수 없기 때문에 작업 지점을 기록해야하며,  
급여나 은행 이자 시스템의 경우 특정 일 (7월, 오늘, 2020년, 등등)에 했던 처리를 또 하는 중복 불상사도 막아야하는 이유가 있습니다.  

## 구현 종류
* 메타 테이블 DB / 운영 테이블 DB 자체를 분리
* 테이블 → 테이블 배치
* 엑셀 → 테이블 배치
* 테이블 → 엑셀 배치
* 웹 API (id) → 테이블 배치
* 스케쥴 (크론식) 기반 실행
* 웹 핸들 기반 실행

## 버전 및 의존성
* Spring Boot 3.3.1
* Spring Batch 5.X
* Spring Data JPA - MySQL
* JDBC API - MySQL
* Lombok
* Gradle-Groovy
* Java 17 ~

## 배치 동작방식 배치에서 중요하게 볼 부분

![1](https://github.com/user-attachments/assets/e8a6452e-9a22-44db-963b-e236bbaeb2d1)  

“배치”는 데이터를 효율적으로 빠르게 처리하는 것도 중요하지만, 더 중요하게 생각하는 부분은 아래와 같다고 생각합니다.  

* 내가 하고 있는 작업을 어디 까지 했는지 계속해서 파악
* 이미 했던일을 중복해서 하지 않게 파악 (배치는 보통 주기적으로 스케쥴러에 잡혀 실행되기 때문에)

즉, 위와 같이 중복이나 놓치는 부분을 파악하기 위해 “기록” 하는 부분이 아주 중요합니다. (앞에서 말한 내용과 같이 기본적인 상황을 지킨 이후에 속도를 높이는 것이 중요합니다.)

## 배치의 흐름
위 모식도를 통해  

* 읽어오기
* 처리하기
* 쓰기

를 읽어올 데이터가 없을때 까지 무한 반복합니다.  

![1](https://github.com/user-attachments/assets/fb23885f-6367-4c45-b6c7-e81815fd09a3)  

왜 한 번에 다 읽지 않을까?  

한 번에 많은 양을 읽어 버리면 메모리에 올리지 못하거나, 실패 했을때 위험성이 크고 속도적인 문제도 발생하기 때문에 대량의 데이터를 끊어서 읽게 된다.  
예를 들어, 2개의 방이 있고 사람이 좌측 방에 있는 책을 외워 나온 뒤 우측 방의 빈 공책에 옮겨적으라는 일이 주어지면, 외울 수 있는 능력과 잊어 버렸을때의 리스크를 고려하기 때문입니다.  

## 위와 같은 동작을 기록하기 위해 : 메타 데이터
그럼 결국 위 작업을 진행하면서 데이터를 끊어서 읽어야하고, 했던 배치 작업을 주기적으로 또 해야하는데 언급 했던 문제들을 해결하기 위해선 기록이 필요합니다.  
배치에서는 내가하고 있는 작업, 했던 작업을 기록하는 테이블을 “메타 데이터”이라고 호칭합니다.  

## 배치 활용 상황 : 메타 데이터 존재 이유
* 은행 이자 시스템 (언제나 등장하는 예시) : 매일 자정 전일 데이터를 기반으로 이자를 계산하여 이자 지급을 수행
* 오늘 자정 기준에 대한 데이터만 처리해야하고 처리 했던 계좌를 또 처리하면 안되며 (중복 지급시 큰일), 10분이라는 빠른 시간 동안 처리 후 은행 영업에 차질이 없도록 만들어야 한다.
* 주기별 회원 권한 조정 : 구독 서비스와 같이 주기별로 권한 변경
* 급여
* 주기별 보고서
* 데이터 삭제

## 스프링 배치의 내부 구조도

![1](https://github.com/user-attachments/assets/dba31dc4-4738-4215-9b0a-329cf345a449)  

JobLauncer : 하나의 배치 작업(Job)을 실행 시키는 시작점  
Job : “읽기 → 처리 → 쓰기” 과정을 정의한 배치 작업  
Step : 실제 하나의 “읽기 → 처리 → 쓰기” 작업을 정의한 부분으로, 1개의 Job에서 여러 과정을 진행할 수 있기 때문에 1 : N의 구조를 가진다.  
ItemReader : 읽어오는 부분  
ItemProcessor : 처리하는 부분  
ItemWriter : 쓰는 부분  
JobRepository : 얼만큼 했는지, 특정 일자 배치를 이미 했는지 “메타 데이터”에 기록하는 부분  

## 참고자료
https://docs.spring.io/spring-batch/reference/domain.html  
https://www.youtube.com/watch?v=MNzPsOQ3NJk&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=2  
https://www.youtube.com/watch?v=AdB6Vtgd2DA&list=PLJkjrxxiBSFCaxkvfuZaK5FzqQWJwmTfR&index=2  

---
# 20240930
# Resilience4J retry, bulk head, actuator

## retry
Resilience4J의 retry 모듈은 실패한 요청을 재시도하는 기능을 가지고 있다.  
retry의 기본적인 우선 순위는 circuit breaker가 실행된 이후 실행되기 때문에 fallbackmethod를 등록한 서킷의 경우 처리하지 않아야 하며 만약 fallbackmethod 동작전 retry를 실행시키고 싶은 경우 내부 config 설정을 통해 우선 순위(Order)를 변경하면 된다.  

## 특정 함수에 retry 패턴 적용
실패가 발생하고 그것을 필수적으로 재시도해야하는 Controller, Service단의 메소드 상단에 retry 어노테이션 선언을 통해 실패 상황을 재시도 할 수 있다.  

* retry 어노테이션 방식 적용
`@Retry(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름")`

* 예시
```java
@Retry(name = "MainControllerMethod1", fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}
```

## name에 대한 retry 설정  
retry를 적용할 특정 name 메소드에 대해 retry 설정을 적용하는 방법  

* application.properties
```
resilience4j.retry.instances.특정할이름.base-config=설정셋
```

## retry 설정
application.properties에 retry 모듈이 제공하는 메소드 설정을 진행한다.  
이때 특정한 이름에 대해서 변수 설정을 진행해 인스턴스 별로 다른 retry 패턴을 지정할 수 있다.  

* 변수에 대한 특정한 이름 설정
```
resilience4j.retry.configs.설정셋.메소드들=값

#예시
resilience4j.retry.configs.default.max-attempts=10
```

* 재요청 설정
```
#재요청 시도 횟수
resilience4j.retry.configs.default.max-attempts=3


#재요청 간격
resilience4j.retry.configs.default.wait-duration=3000ms
```

* 예외 처리 (retry에 포함) (만약 ignore에도 포함되어 있는 경우 ignore 우선)
```
예외 처리 (retry에 포함) (만약 ignore에도 포함되어 있는 경우 ignore 우선)
```

* 예외 처리 (retry에 포함 안함)
```
resilience4j.retry.configs.default.ignore-exceptions[0]=java.io.IOException
```

## fallbackmethod 설정
fallbackmethod는 설정한 재시도 최대 횟수가 초과된 이후 실행할 메소드로 어노테이션 인자를 통해 설정할 수 있다.  

```java
@Retry(name = "MainControllerMethod1", fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}


private String 실패시수행할메소드이름(Throwable throwable) {

    return throwable.getMessage();
}
```
유의할 점으로 실패시 수행할 메소드에 매개변수로 `Throwable throwable`는 필수이며 만약 기존 함수에 매개변수가 있다면 그거도 같이 넣어줘야한다.  
위의 예시에서 ` mainP(String a, String b)` 이렇게 매개 변수가 있다면 `실패시수행할메소드이름(String a, String b, Throwable throwable)` 이렇게 써야한다.  

## bulk head
Resilience4J가 제공하는 bulk head 모듈은 동시 요청을 제한하는 기능을 가진다.  
이때 bulk head 모듈은 두가지 타입을 제공하며 아래와 같다.  

* bulkhead (semaphore)
세마포어 알고리즘을 통해 공유 자원 접근을 제한하는 방식

* thread-pool-bulkhead (fixed thread pool)
고정된 사이즈의 스레드를 지정하는 방식

하나의 메소드에 대해 위의 두 bulkhead 타입 중 하나를 선택하여 구현해야 한다.  

## 각 타입이 존재하는 이유

@Async 어노테이션이 선언된 비동기 메소드와 bulkhead(semaphore) 방식의 자원 제한을 사용할 경우 요청에 의해 생성된 스레드는 재사용되지 않고 계속적으로 증가하는 결과를 가진다고 합니다. (세마포어 카운터가 0이 되어도 새로운 스레드를 생성하기 때문에 bulk head 기능을 못 함)  
따라서 비동기 메소드의 경우 해당 메소드에 대해 전체 스레드 개수를 제한하는 thread-pool-bulkhead 방식을 사용하는 것이 좋다고 합니다.  

블로그 참고 : https://dev.to/gabrielaramburu/bulkhead-pattern-semaphore-vs-threadpool-226p  

## 특정 함수에 bulk head 패턴 적용
동시 요청 제한을 진행해야하는 메소드에  

* bulkhead (semaphore) 방식
```java
@Bulkhead(name = "특정할이름", type = Bulkhead.Type.SEMAPHORE, fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}
```

* thread-pool-bulkhead (fixed thread pool)
```java
@Bulkhead(name = "특정할이름", type = Bulkhead.Type.THREADPOOL, fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}
```

## name에 대한 bulk head 설정
application.properties에서 설정  

* bulkhead (semaphore) 방식
`resilience4j.bulkhead.instances.특정할이름.base-config=설정셋`

* thread-pool-bulkhead (fixed thread pool)
`resilience4j.thread-pool-bulkhead.instances.특정할이름.base-config=설정셋`

## bulk head 설정

application.properties에서 설정  

* bulkhead (semaphore) 방식
```
#동시 요청 세마포어 카운터 수
resilience4j.bulkhead.configs.default.max-concurrent-calls=1


#동시 요청 초과시 웨이팅 시간 (초과되면 exception 발생)
resilience4j.bulkhead.configs.deafult.max-wait-duration=1000ms
```

* thread-pool-bulkhead (fixed thread pool)
```
#최대 스레드 풀
resilience4j.thread-pool-bulkhead.configs.defatul.max-thread-pool-size=10


#기본 스레드 풀
resilience4j.thread-pool-bulkhead.configs.default.core-thread-pool-size=5


#스레드풀 초과시 요청이 기다릴 대기큐 크기
resilience4j.thread-pool-bulkhead.configs.default.queue-capacity=50
```

## thread-pool-bulkhead 오류에 대한 GitHub Issue
“errorThreadPool bulkhead is only applicable for completable futures”  
토이프로젝트 수준에서 `thread-pool-bulkhead` 설정에서 오류가 있는데 이슈가 올라온게 있다. 실사용할때는 잘 찾아보고 사용하기  

https://github.com/resilience4j/resilience4j/issues/826  

## fallbackmethod
```java
@Bulkhead(name = "특정할이름", type = Bulkhead.Type.SEMAPHORE, fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}

private String 실패시수행할메소드이름(Throwable throwable) {

    return throwable.getMessage();
}
```

## actuator를 통한 Resilience4J 서킷 상태 확인
Spring Boot Actuator를 통한 Resilience4J가 적용된 마이크로서비스의 서킷 상태를 확인할 수 있다.  

## actuator 의존성 추가

* build.gradle
```
dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

## actuator 엔드포인트 활성화
application.properties에서 설정을 진행한다.  

* actuator 관련
```
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.health.circuitbreakers.enabled=true
```

* resilience4j circuit breaker 관련
`resilience4j.circuitbreaker.configs.default.register-health-indicator=true`

## actuator 엔드포인트
`/actuator/health`  

![스크린샷 2024-09-14 090020](https://github.com/user-attachments/assets/90c91a7a-da04-4e8c-9c42-5cb048891b67)  

actuator를 등록하면 설정해둔 Resilience4J 상태를 확인할 수 있다.  

## 참고
https://www.youtube.com/watch?v=klfx8Xq-cq4&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=7  
https://www.youtube.com/watch?v=11VlEZcvZ-g&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=9  
https://www.youtube.com/watch?v=mQb6exPfnHk&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=10  

---
# 20240914
# Resilience4J 슬라이딩 윈도우, 서킷브레이커 구현

## 슬라이딩 윈도우 기법  
슬라이딩 윈도우 기법을 통해 지정해둔 사이즈의 슬라이딩 윈도우 터널을 생성하고 내부에서 실패 및 지연 개수를 측정한다.  
![1](https://github.com/user-attachments/assets/00364782-afa9-4d1b-be5a-056f03521511)  

## circuit breaker 구현 구조
![1](https://github.com/user-attachments/assets/583c431c-aadd-410b-b08a-8d075e31f9cf)  
Service2에서 Service1을 호출하는 상황에서 Service2에 circuit을 open할 Resilience4J의 circuit breaker를 구현하는 방법  

## 특정 함수에 circuit breaker 패턴 적용
마이크로서비스의 내부 로직 중 지연 또는 실패가 발생할 수 있는 Controller, Service단에 circuit breaker를 적용하여 해당 상황이 발생하였을때 대응할 수 있도록 구성한다.  

사용할 함수에 circuit breaker 어노테이션 방식 적용  
`@CircuitBreaker(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름")`

예시  
```java
@CircuitBreaker(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}
```

## name에 대한 circuit breaker 설정
circuit breaker를 적용할 특정 name 메소드에 대해 circuit breaker 설정을 적용하는 방법  

* application.properties
```
resilience4j.circuitbreaker.instances.특정할이름.base-config=설정셋
```

## circuit breaker 설정
application.properties에 circuit breaker 모듈이 제공하는 메소드 설정을 진행한다.  

이때 특정한 이름에 대해서 변수 설정을 진행해 인스턴스 별로 다른 circuit breaker 패턴을 지정할 수 있다.  

* 변수에 대한 특정한 이름 설정
```
resilience4j.circuitbreaker.configs.설정셋.메소드들=값

#예시
resilience4j.circuitbreaker.configs.default.failure-rate-threshold=10
```

* 슬라이딩 윈도우 공통 사항
```
#실패 및 지연을 체크할 슬라이딩 윈도우 타입 (개수 기반)
resilience4j.circuitbreaker.configs.default.sliding-window-type=count_based


#슬라이딩 윈도우 크기
resilience4j.circuitbreaker.configs.default.sliding-window-size=5
```

* 실패에 대한 설정
```
#서킷을 오픈할 실패 비율 (실패 수 / 슬라이딩 윈도우 크기) (퍼센트)
resilience4j.circuitbreaker.configs.default.failure-rate-threshold=10


#서킷을 오픈하기 위해 최소 실패 수 (슬라이딩 윈도우를 다 채우지 못했지만 최소값을 설정 가능)
resilience4j.circuitbreaker.configs.default.minimum-number-of-calls=5
```

* 지연에 대한 설정
```
#서킷을 오픈할 지연 비율 (지연 수 / 슬라이딩 윈도우 크기)
resilience4j.circuitbreaker.configs.default.slow-call-rate-threshold=10


#지연으로 판단할 시간
resilience4j.circuitbreaker.configs.default.slow-call-duration-threshold=3000ms
```

* half open 상태 설정

```
#half open 상태에서 다른 상태로 전환하기 위한 판단 수
resilience4j.circuitbreaker.configs.default.permitted-number-of-calls-in-half-open-state=10


#half open 상태 유지 시간 (만약 0이면 위에서 설정한 값 만큼 수행 후 다음 상태로 전환)
resilience4j.circuitbreaker.configs.default.max-wait-duration-in-half-open-state=0


#open 상태에서 half open으로 전환까지 기다리는 시간
resilience4j.circuitbreaker.configs.default.wait-duration-in-open-state=600000ms


#open 상태에서 half open 으로 자동 전환 (true시 일정 시간이 지난 후 자동 전환)
resilience4j.circuitbreaker.configs.default.automatic-transition-from-open-to-half-open-enabled=true


#상태 체크 표시 (actuator용)
resilience4j.circuitbreaker.configs.default.register-health-indicator=true
```

* 예외 처리 (아래 예외 또한 실패로 받아드리기 때문에 처리하지 않으면 실패수에 포함 됨)
```
resilience4j.circuitbreaker.configs.default.ignore-exceptions[0]=java.io.IOException
resilience4j.circuitbreaker.configs.default.ignore-exceptions[1]=java.util.concurrent.TimeoutException
resilience4j.circuitbreaker.configs.default.ignore-exceptions[2]=org.springframework.web.client.HttpServerErrorException
```

## fallback 메소드 설정

```java
@CircuitBreaker(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름")
@GetMapping("/")
public String mainP() {

    return rest1Comp.restTemplate1().getForObject("/data", String.class);
}

private String 실패시수행할메소드이름(Throwable throwable) {

    return throwable.getMessage();
}
```

fallback 메소드는 circuit breaker가 적용된 메소드가 가지는 인자를 필수적으로 가져야 합니다.  

## 결과  

![스크린샷 2024-09-13 135856](https://github.com/user-attachments/assets/1127479f-abaf-489c-b568-68b87d84f0ff)  

일정 횟수 이상 실패하면 서킷이 오픈되도록 설정되어있으므로 첫 몇번의 실패는 그냥 요청실패 화면이 뜬다.  

![스크린샷 2024-09-13 135907](https://github.com/user-attachments/assets/365a95c4-bdd2-4cfb-b46d-cc269b879b12)  
일정 횟수 이상 실패하게 되면 서킷이 오픈되었다는 응답으로 바꿔서 오게된다.  

## 참고  
https://www.youtube.com/watch?v=vfDXRZqrzSs&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=5  
https://www.youtube.com/watch?v=U28Q3kDwcg4&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=7  

---
# 20240913
# 프로젝트 Resilience4J사용서버, 호출서버 구축

## resilience4j 서버

## 버전
스프링 부트 버전 : 3.2.0  
언어 : Java  
의존성 관리 도구 : gradle (groovy)  

## 기본 모듈 : build.gradle
spring web  
lombok  
resilience4j  

```
plugins {
  id 'java'
  id 'org.springframework.boot' version '3.2.0'
  id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
  sourceCompatibility = '17'
}

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

ext {
  set('springCloudVersion', "2023.0.0")
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}

tasks.named('test') {
  useJUnitPlatform()
}
```

추가 모듈 이것 추가해야 어노테이션 기반으로 resilience4j 사용가능  

```
dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-aop'

    implementation 'io.github.resilience4j:resilience4j-all'
}
```

## 컨트롤러

```java
@Controller
@ResponseBody
public class MainController {

		private Rest1Comp rest1Comp;

    @Autowired
    public MainController(Rest1Comp rest1Comp) {

        this.rest1Comp = rest1Comp;
    }

    @GetMapping("/")
		public String mainP() {

        return rest1Comp.restTemplate1().getForObject("/data", String.class);
    }
}
```

## RestTemplate 생성 : 다른 서비스 호출을 위한 API Client

```java
@Component
public class Rest1Comp {

    @Bean
    public RestTemplate restTemplate1() {

        return new RestTemplateBuilder().rootUri("http://localhost:9000")
                .build();
    }
```

## 호출서버

Service1 API 서버 구축  
Spring Boot 3.2.0  

## DataController  

```java
@Controller
@ResponseBody
public class DataController {

    @GetMapping("/data")
    public String dataP() {

        String nowTime = String.valueOf(LocalDateTime.now());

        try {

            Thread.sleep(10000);
        } catch (InterruptedException e) {

            throw new RuntimeException(e);
        }

        return nowTime;
    }
}
```

* application.properties
`server.port=9000`

이렇게 호출서버쪽에서 10초를 기다린후 응답이 넘어오도록 세팅했고  
resilience4j 에서도 호출서버쪽의  api를 사용할수 있도록 셋팅완료 했다  

## 참고
https://www.youtube.com/watch?v=f7-t8o9NbFI&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=3  
https://www.youtube.com/watch?v=mN8RtsKAaBI&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=5  

---
# 20240912
# Resilience4J 장애대응방법, 모듈

## MSA 장애 상황과 대응
MSA는 여러개의 마이크로 서비스가 서로 호출을 하며 하나의 시스템을 이룬다. 이때 각각의 MS는 무응답, 지연, 실패와 같은 상황을 발생할 수 있습니다.  
위 상황이 발생할때 해당 부분 MS에 circuit을 open하여 일시적으로 다른 작업을 처리하도록하는 MSA 장애 대응을 진행해야 한다.  
스프링 프레임워크 기반으로 MSA를 구축할때 위와 같이 서킷 방식의 장애 대응 솔루션을 제공하는 프레임워크는 Netflix Hystrix와 Resilience4J가 존재한다.  

## Resilience4J
Netflix Hystrix의 지원이 중단되면서 Spring Cloud에서 Hystrix를 계승한 Resilience4J 서킷 브레이크 프레임워크를 제공한다.  
Resilience4J는 자바용 내결함성 라이브러리로 하나의 서비스에서 발생할 수 있는 장애 상황에서 그것을 대처할 수 있는 여러 솔루션을 제공한다.  

공식사이트 : https://resilience4j.readme.io/  

## 예시
* 장애 상황이란?
* 실패 : 요청 했는데 응답이 안옴 (health check로 금방 알 수 있음)
* 지연 : 요청 했는데 평소보다 늦게 응답이 옴 (응답이 왔기 때문에 파악하기 힘듬)

![1](https://github.com/user-attachments/assets/a826b3cd-9169-4603-aefd-673c9c14e637)  
캐시 DB를 활용해서 효율적으로 쓰고있다가 이 서버와 문제가 생겨서 실패하거나 지연이 상당히 길어진다. 이렇게되면 이를 캐치해서 RDB쪽으로 연결을 open해서 사용하게 만들어 준다.  

## 참고  
https://www.youtube.com/watch?v=UpwJOImeYcA&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=1  

## Resilience4J 제공 모듈

공식문서 : https://docs.spring.io/spring-cloud-circuitbreaker/reference/index.html  

* circuit-breaker : 회로 차단기
	* 실패
	* 지연
* fall-back : 실패시 동작
* retry : 자동 재시도
* bulk-head : 동시 실행 제한
* rate-limiter : 속도(성능) 제한
* time-limiter : 시간 초과 제한
* cache : 결과 캐싱

## circuit-breaker
Resilience4J의 핵심 모듈로 장애 또는 지연 상황에서 circuit을 일시적으로 오픈하는 기능을 제공한다.  

서킷을 오픈하는 조건은 아래와 같다.   
* 실패 : 호출 후 특정 비율 이상 실패
* 지연 : 호출 후 특정 비율 이상 지연

서킷은 아래와 같은 유한 상태를 가진다.  

![1](https://github.com/user-attachments/assets/583c431c-aadd-410b-b08a-8d075e31f9cf)  

(service2가 service1을 호출하는 상황이라고 가정하고)  

![1](https://github.com/user-attachments/assets/fb0b5d11-c5ec-44a1-93ab-840cbfe3b3dd)  

CLOSED : 평상시 상태  
OPEN : 평상시 상태에서 사용자가 설정한 임계치 이상의 지연율 및 실패율이 달성되면 회로를 끊어버린 상태  
HALP_OPEN : OPEN 상태에서 설정한 시간이 지난 후 HALF_OPEN 상태로 전환된다. HALF_OPEN 상태는 CLOSED 또는 OPEN 상태로 전환을 판단한다.  

## fall-back
circuit-breaker가 실패 또는 지연 상황 발생으로 circuit을 open 할 경우 대비책으로 동작할 메소드 설정.  

## retry
실패한 요청을 일정 시간 이후에 재시도하는 모듈.  

## bulk-head
병렬 실행 제한을 위한 모듈.  

## rate-limiter
마이크로 서비스 내부 실행 중 일정 비율 이상의 부하를 막기 위한 속도 제한 모듈.  

## time-limiter
마이크로 서비스 내부 실행 중 일정 시간 이상의 지연을 막기 위한 시간 제한 모듈.  

## cache
요청에 대한 결과 저장(캐싱)을 위한 모듈.  

## 읽어보면 좋을 자료
올리브영 테크블로그의 Circuitbreaker를 사용한 장애 전파 방지  
https://oliveyoung.tech/blog/2023-08-31/circuitbreaker-inventory-squad/  

![1](https://github.com/user-attachments/assets/73c3bd62-67f7-4e8d-8eb2-4377fb6671d6)  

올리브영의 예시인데 많은 서비스가 이런식으로 구현되어있을 것이다. 레디스를 캐시로 사용하는 효율을 높이는 DB 사용법

![2](https://github.com/user-attachments/assets/6dc65135-6d55-4816-b5b7-20bdc9a9a227)  

다만 이 레디스도 문제가 생길 수 있다 연결이 끊긴다던지 어떠한 원인에 의해 그냥 RDB에서 찾는것보다 느리다던지  

![3](https://github.com/user-attachments/assets/8c6938d4-889f-476b-b784-494525d7235b)  

위 문제를 해결하기 위해서 Resilience4J를 사용할 수 있다 이를 앞단에 두어서 레디스에 문제가 생기면 바로 RDB쪽으로 우회해주는것이다.  

G마켓 기술 블로그의 Fault Tolerance  
https://dev.gmarket.com/86  

## 참고
https://www.youtube.com/watch?v=EL2XEh6l_Rc&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ&index=3  


---
# 20240911
# 게이트웨이 라우터추가, 글로벌필터, 지역필터

## 게이트웨이의 특성
스프링 클라우드 게이트웨이는 퍼블릭 엔드포인트에서 게이트웨이 역할을 수행하기 때문에 서비스를 시작한 후 항상 가동 상태여야 한다.  

## 라우팅의 추가와 삭제
항상 가동 상태로 유지되어야 하는 게이트웨이 특성상 서버 스크립트를 중지 후 코드를 수정하고 재배포하면 서비스의 실시간성을 보장하지 못한다.  
스프링 클라우드 게이트웨이의 경우 가동 중 새로운 비즈니스 로직(경로)이 추가될 경우 해당 주소에 대한 라우팅을 즉시 추가하고 삭제할 수 있는 여러 기능을 제공한다.  

## Actuator
Actuator는 스프링 어플리케이션의 기능을 엔드 포인트로 제공하는 의존성이다. 이 Actuator를 활용하여 가동중인 스프링 클라우드 게이트웨이에 새로운 라우팅을 추가하고 삭제할 수 있다.  

## 프로젝트 생성과 의존성 추가
* 필수 의존성
* Gateway
* Spring Boot Actuator

필히 실제로 사용할때는 시큐리티를 추가해서 외부에서 Actuator 접근못하게 설정해야함  

## Actuator 설정
* application.properties
```
management.endpoint.gateway.enabled=true
management.endpoints.web.exposure.include=gateway
```
게이트 웨이쪽 Actuator만 사용하도록 설정  

## 라우팅 명령어
우와 같이 세팅하면 외부에서 게이트웨이 쪽으로 http요청을 날려서 라우터를 추가하고 삭제할 수 있다.  

* 존재하는 라우팅 확인
`GET : /actuator/gateway/routes`

* 라우터 추가
`POST : /actuator/gateway/routes/{id}`

POST Body에 JSON 타입으로 데이터를 추가해야 한다.  

* 리프래시
`POST : /actuator/gateway/refresh`
추가 삭제후 꼭 이 api를 보내줘야 등록이 완료된다.

* 라우트 제거
`DELETE : /actuator/gateway/routes/{id}`

* 특정 라우트 확인
`GET : /actuator/gateway/routes/{id}`

* 글로벌 필터 목록
`GET : /actuator/gateway/globalfilters`

* 특정 라우터 필터 목록
`GET : /actuator/gateway/routefilters/{id}`

## 라우팅 추가 실습

* 현재 라우팅 목록 확인
`GET : /actuator/gateway/routes`

* 라우팅 추가
`POST : /actuator/gateway/routes/{이름}`

```
{
    "predicate": "Paths: [/ms1/**]",
    "filters": [],
    "uri": "http://localhost:8081",
    "order": 0
}
```

* 추가 후 refresh
`POST : /actuator/gateway/refresh`

## 참조
https://www.youtube.com/watch?v=G2D3m8qhNiI&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=13  

## 글로벌 필터

![1](https://github.com/user-attachments/assets/b2c8d81c-c033-48c5-afe4-6aad2b5ecd9e)  

스프링 클라우드 게이트웨이에서 글로벌 필터는 모든 라우팅에 대해서 적용되는 필터이다. 따라서 필터만 구현하면 특별한 설정 없이 적용된다.  
클라이언트의 요청은 필터 → 마이크로서비스 → 필터 형태로 이동되며 같은 필터라도 마이크로서비스를 접근하기 이전이면 pre, 이후면 post라고 명명한다.  
각각의 필터는 Order 값을 가질 수 있으며 pre 필터의 경우 Order 값이 작을수록 빠르게 동작하며, post 필터의 경우 Order 값이 작을수록 늦게 동작한다.  

## 글로벌 필터 작성

```java
@Component
public class G1Filter implements GlobalFilter, Ordered {


    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        System.out.println("pre global filter order -1");

        return chain.filter(exchange)
                .then(Mono.fromRunnable(() -> {

                    System.out.println("post global filter order -1");
                }));
    }

    @Override
    public int getOrder() {

        return -1;
    }
}
```
-1 순서로 등록한것이고 `System.out.println("pre global filter order -1");`이 부분이 pre, 아래 `return`이후가 post 부분이다.  

## Order 설정시
각각의 마이크로서비스에 요청을 전달하는 라우팅 테이블이 Order 0으로 설정된 경우가 많기 때문에 필터의 경우 음수 설정이 지향된다.  

## 참조
https://www.baeldung.com/spring-cloud-custom-gateway-filters  
https://www.youtube.com/watch?v=TgbUGQ-jO2I&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=14  

## 지역 필터
스프링 클라우드 게이트웨이에서 지역 필터는 특정 마이크로서비스 라우팅에 대해서만 동작을 진행하는 필터이다.  

## 프로젝트 필수 의존성

* Gateway
* Lombok

## 지역 필터 작성

```java
@Component
public class L1Filter extends AbstractGatewayFilterFactory<L1Filter.Config> {

    public L1Filter() {

        super(Config.class);
    }


    @Override
    public GatewayFilter apply(Config config) {

        return (exchange, chain) -> {

            if (config.isPre()) {
                System.out.println("pre local filter 1");
            }

            return chain.filter(exchange)
                    .then(Mono.fromRunnable(() -> {

                        if (config.isPost()) {

                            System.out.println("post local filter 1");
                        }
                    }));
        };
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Config {
        private boolean pre;
        private boolean post;
    }
}
```
지역 필터의 경우 첫번째 `return`이 pre, 두번째 `return`이 post이다.  

## 특정 라우팅에 지역 필터 등록  
* application.properties
```
spring.cloud.gateway.routes[0].filters[0].name=L1Filter
spring.cloud.gateway.routes[0].filters[0].args.pre=true
spring.cloud.gateway.routes[0].filters[0].args.post=true
```

* appliction.yml
```
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: ms1
          uri: http://localhost:8081
          predicates:
            - Path=/ms1/**
					filters:
						- name: L1Filter
							args:
								pre: true
								post: true
        - id: ms2
          uri: http://localhost:8082
          predicates:
            - Path=/ms2/**
```

* config 클래스
```java
@Configuration
public class RouteConfig {

    @Bean
    public RouteLocator ms1Route(RouteLocatorBuilder builder) {

        return builder.routes()
                .route("ms1", r -> r.path("/ms1/**")
												.filters(f -> f.filter(L1Filter.apply(new L1Filter.Config(true, true))))
                        .uri("http://localhost:8081")
								)
                .route("ms2", r -> r.path("/ms2/**")
                        .uri("http://localhost:8082")
								)
                .build();
    }
}
```

## 참조
https://www.youtube.com/watch?v=Tg5_6XW61sQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=14  

---
# 20240905
# 게이트웨이 라우팅 설정, Eureka로드벨런싱

![1](https://github.com/user-attachments/assets/f149cca3-64cf-4962-8959-4e3d22fd05dc)  

## 라우트 설정 조건들

* 시간별로 서버 분할
```
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]  
```

* HTTP 헤더로 분할
```
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

* HTTP 메소드로 분할
```
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

등등 여러 방법이 있다 공식 페이지를 참조  
https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories  

## application.properties를 통한 경로 설정

```
server.port=8080


spring.cloud.gateway.routes[0].id=ms1
spring.cloud.gateway.routes[0].predicates[0].name=Path
spring.cloud.gateway.routes[0].predicates[0].args.pattern=/ms1/**
spring.cloud.gateway.routes[0].uri=http://localhost:8081


spring.cloud.gateway.routes[1].id=ms2
spring.cloud.gateway.routes[1].predicates[0].name=Path
spring.cloud.gateway.routes[1].predicates[0].args.pattern=/ms2/**
spring.cloud.gateway.routes[1].uri=http://localhost:8082
```

## application.yml를 통한 경로 설정

```
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: ms1
          uri: http://localhost:8081
          predicates:
            - Path=/ms1/**
        - id: ms2
          uri: http://localhost:8082
          predicates:
            - Path=/ms2/**
```

## Config 클래스를 활용한 경로 설정

```
@Configuration
public class RouteConfig {

    @Bean
    public RouteLocator ms1Route(RouteLocatorBuilder builder) {

        return builder.routes()
                .route("ms1", r -> r.path("/ms1/**")
                        .uri("http://localhost:8081"))
                .route("ms2", r -> r.path("/ms2/**")
                        .uri("http://localhost:8082"))
                .build();
    }
}
```

## 참조  
https://www.youtube.com/watch?v=MAy3QTUS5VM&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=11  

## 게이트웨이와 Eureka 서버 연동
Eureka 서버는 각각의 비즈니스 로직 처리를 담당하는 스프링 부트 어플리케이션 목록들을 가지고 있다.  
MSA를 구성하는 요소는 자동으로 오토 스케일링되기 때문에 새로 생긴 서버의 IP를 게이트웨이가 알지 못한다. 따라서 Eureka 서버가 해당 목록들을 관리하며 Gateway에게 전달한다.  

## 스프링 클라우드 게이트웨이 Eureka 클라이언트 설정

* build.gradle
```
ext {
  set('springCloudVersion', "2022.0.4")
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

## application.properties 설정

```
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true
eureka.client.service-url.defaultZone=http://아이디:비밀번호@아이피:8761/eureka
```

## 게이트웨이 라우팅 유레카 로드 밸런싱

* application.properties
```
spring.cloud.gateway.routes[0].id=ms1
spring.cloud.gateway.routes[0].predicates[0].name=Path
spring.cloud.gateway.routes[0].predicates[0].args.pattern=/ms1/**
spring.cloud.gateway.routes[0].uri=lb://MS1
```
유레카로 등록할때 두개의 서버를 MS1의 이름으로 등록해두고 게이트웨이에서 `spring.cloud.gateway.routes[0].uri=lb://MS1` 이런식으로 적용하게되면  
해당 주소로 요청이 들어갈때 자동으로 서버1 > 서버2 > 서버1 > 서버2 이렇게 돌아가면서 요청이 들어가게된다.  

## 참조
https://www.youtube.com/watch?v=bL7bakWi6Vg&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=12  

---
# 20240904
# Eureka 클라이언트 설정, 스프링 클라우드 게이트웨이 기초
![1](https://github.com/user-attachments/assets/f149cca3-64cf-4962-8959-4e3d22fd05dc)  

## Eureka 클라이언트란
MSA를 구성하는 요소들 중 Eureka 서버에서 모니터링 및 관리를 원하는 요소를 Eureka 클라이언트 설정을 진행해서 등록할 수 있다.  

## Eureka 클라이언트 설정을 위한 의존성 추가
* 필수 의존성
* Eureka Discovery Client

* build.gradle
```gradle
ext {
  set('springCloudVersion', "2022.0.4")
}

dependencies {

  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```
ext,dependencyManagement는 없으면 추가해줘야한다.  

## 어노테이션 등록
스프링 부트 메인 클래스에 Eureka 클라이언트로 동작하기 위한 어노테이션을 등록해야 한다.  
`@EnableDiscoveryClient`  

## Eureka 서버와 연결
application.properties 변수 설정을 통해 Eureka 서버에 등록할 수 있다.  

```properties
server.port=8080
spring.application.name=ms1


eureka.client.register-with-eureka=true #유레카 서버에 등록할지 여부
eureka.client.fetch-registry=true #유레카 서버의 정보를 가져올지 여부
eureka.client.service-url.defaultZone=http://아이디:비밀번호@아이피:8761/eureka #유레카 서버 주소
```

## 참고
https://www.youtube.com/watch?v=h1c3Rqt26kQ  

## Spring Cloud Gateway란?
스프링 클라우드 게이트웨이는 MSA 가장 앞단에서 클라이언트들로 부터 오는 요청을 받은 후 경로와 조건에 알맞은 마이크로서비스 로직에 요청을 전달하는 게이트웨이이다.  
게이트웨이는 개념적으로는 아주 단순하지만 가장 앞단에서 무중지 상태로 모든 요청을 받아야하기 때문에 설정하기에 까다롭다.  

## Spring Cloud Gateway의 특성
기존에 제작했던 스프링 부트, Eureka, Config와 같은 서비스들을 블로킹 기반으로 모두 톰캣 엔진을 사용했다.  
하지만 게이트 웨이의 경우 비즈니스 로직 처리 보단 단순하게 지나가는 통로 즉, I/O 처리를 중점적으로 진행하기 때문에 논 블로킹 방식으로 동작하는 WebFlux와 네티엔진을 사용한다.  
WebFlux는 기존에 스프링 부트에서 사용했던 JPA와 같은 블로킹 방식의 의존성들을 모두 사용하지 못하기 때문에 구현에 앞서 많은 학습이 필요하다.  

## 프로젝트 생성과 의존성 추가
* 필수 의존성
* Gateway

## 게이트 웨이 설정 방식
* 설정 파일 방식
    * application.properties
    * application.yml
* 클래스 방식

## 참고
https://www.youtube.com/watch?v=AoPuwW2uz5s  

---
# 20240822
# JetBrains Space Config 리포지토리, Eureka 서버 구축

## JetBrains Space
깃허브에 프라이빗 레포말고 다른 괜찮은 서비스 있어서 추천겸 정리  
https://www.jetbrains.com/space/  

JetBrains사가 제공하는 Space는 Git 서비스, CI/CD, 온라인 IDE, 이슈 트래킹, 채팅등을 지원하는 Developmtent 플랫폼이다.  

## Space Repository를 Config 서버 저장소로 사용하는 방법
Space에도 Repository가 존재하는데 이 Repository를 Config 저장소로 사용할 수 있다.  
연결 방법은 SSH와 비대칭키가 아닌, HTTPS와 Space의 계정 아이디, 비밀번호를 통해 연결을 진행할 수 있다.  

여기에는 따로 비대칭키 저장하는게 없어서 JetBrains 아이디 비밀번호로 설정한다.  

따라서 스프링 Config 서버의 application.properteis 설정은 아래와 같이 진행하면 된다.  

```
spring.cloud.config.server.git.uri=리포지토리HTTPS주소
spring.cloud.config.server.git.username=아이디
spring.cloud.config.server.git.password=비밀번호
```

## 참고
https://www.youtube.com/watch?v=ujgz094STCc  

## Eureka 서버 구축
![1](https://github.com/user-attachments/assets/f149cca3-64cf-4962-8959-4e3d22fd05dc)  

## Eureka 서버의 역할
Eureka 서버는 단순하게 MSA를 구성하는 마이크로 서비스들을 모니터링하는 감시자 서버의 역할을 한다고 볼 수 있지만.  
깊게 보면 현재 존재하는 마이크로 서비스들을 Gateway에게 알려주어 시간과 부하에 따라 유동적으로 스케일 아웃되는 서버들을 모두 가용할 수 있도록 하는 역할을 수행한다.  
따라서 Eureka 서버는 가동되는 서버를 확인 후 Gateway에게 그 목록을 알려주는 역할을 수행한다.  

## 프로젝트 생성과 의존성 추가
* 필수 의존성
* Eureka Server
* Spring Security

## Main 클래스 어노테이션 등록
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

## Eureka 서버 설정
application.properties 또느 application.yml 파일에 설정을 통해 Eureka 서버 생성을 진행할 수 있다.  

```
server.port=8761


eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

## 시큐리티 설정

Eureka 서버는 내부망에서만 존재해야 된다. 하지만 외부망에 구축하거나 내부망에서도 보안을 중요시 해야하기 때문에 스프링 시큐리티 설정을 진행한다.  
시큐리티는 httpBasic 방식 코드를 작성하면 된다  

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth.anyRequest().authenticated());

        http
                .httpBasic(Customizer.withDefaults());


        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {

        UserDetails user1 = User.builder()
                .username("아이디")
                .password(bCryptPasswordEncoder().encode("비밀번호"))
                .roles("ADMIN")
                .build();


        return new InMemoryUserDetailsManager(user1);
    }
}
```

## 참고
https://www.youtube.com/watch?v=OXDwOYxKvWA&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=7  

---
# 20240820
# config server, config client

![1](https://github.com/user-attachments/assets/fe1744d2-a4d2-4aa6-b464-e76cf9cfc85a)  

## Config 저장소 주소와 접속 private 키 준비
* 리포지토리 주소 : SSH
![1](https://github.com/user-attachments/assets/8b451732-532a-435e-9fd1-1d4545d40d34)


* 비대칭키 중 private 키 내용 가져오기

## 프로젝트 생성과 의존성 추가

* 필수 의존성
* Config Server
* Spring Security

## Main 클래스 어노테이션 등록
`@EnableConfigServer`  

## Config 저장소 연결
application.properties 파일을 통해 Config 저장소를 연결한다.  

```
server.port=9000

spring.cloud.config.server.git.uri=주소
spring.cloud.config.server.git.ignoreLocalSshSettings=true
spring.cloud.config.server.git.private-key=비밀키내용
```

## Config 서버 시큐리티 설정
기본적으로 Config Client와 Config Server간 통신시 내부망을 사용하지만 추가적인 보안을 위해 HttpBasic 보안 설정을 권장한다. 따라서 Security Config 설정을 통해 모든 경로에 대해서 HttpBasic 보안 설정을 진행한다.  

* Security Config 클래스 생성
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth.anyRequest().authenticated());

        http
                .httpBasic(Customizer.withDefaults());


        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {

        UserDetails user1 = User.builder()
                .username("아이디")
                .password(bCryptPasswordEncoder().encode("비밀번호"))
                .roles("ADMIN")
                .build();


        return new InMemoryUserDetailsManager(user1);
    }
}
```
복잡한 유저정보가 필요한게 아니기에 접속 유저 아이디 비번 인메모리로 직접 하나 추가

## 접근 주소
설정 정보 데이터를 얻기 위해 Config Client가 Config Server에 접근하는 주소는 아래와 같다.  

`http://ip:port/저장소이름/저장소환경`  

IP와 포트는 Config Server의 값을 입력해주지만 저장소이름과 저장소환경 경로는 Config Repository에 해당하는 깃허브 리포지토리 내부 파일 명을 넣어야 한다.  
이때 파일명에 대한 주소 변환은 아래와 같다.  

```
이름-환경.properties
이름-환경.yml

/이름/환경
```

## 참고자료
https://www.youtube.com/watch?v=qeQPBImEGec  

## Config 클라이언트란
단순하게 서비스 로직을 수행하는 스프링 부트 어플리케이션이다.  
스프링 부트 어플리케이션에 Config Client 설정을 통해 Config Server에서 보내주는 데이터를 받을 수 있다.  

## 프로젝트에 클라이언트 의존성 추가
* 필수 의존성
* Config Client

```
ext {
  set('springCloudVersion', "2022.0.4")
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-config'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```
스프링스타터에서도 Config Client찾아서 미리보기하면 그레들에 `ext` `dependencyManagement`이런거 새로 추가되어있다. 기존 프로젝트에 Config Client 추가하려면 이런거도 찾아서 복붙해줘야한다.  

## Config 서버와 연결
application.properties 파일을 통해 Config 서버에 연결 가능  

```
spring.application.name=이름
spring.profiles.active=환경
spring.config.import=optional:configserver:http://아이디:비밀번호@아이피:포트
```
이렇게 등록해주면 내가 택한 폴더의 환경변수파일을 그데로 사용한다.  

## 서버로 부터 데이터 받기

* application.properties
```
server.portname=${server.port}
```
그대로 사용하지 않고 새로운 환경변수명에 데이터담으려면 위와같이 사용하면 포트정보가 server.portname에 저장된다.

## 참고
https://www.youtube.com/watch?v=EQ31GAYxqSk  

---
# 20240819
# Config 깃허브 리포지토리

![1](https://github.com/user-attachments/assets/a5928dc2-cb35-47c6-a720-a1be3aca4140)  

## Config 저장소의 종류
Config Server는 데이터를 전달하는 매개체 역할만 수행하고 실제 설정 정보를 담을 영속성의 경우 DB, 파일, Git Service를 사용한다.  

### Config 영속성의 종류
* Git Service
* RDB
* Document NoSQL
* Redis
* File
* Vault
* 등등..

여러 영속성 도구 중 Git Service를 가장 많이 사용한다.  

## 깃허브 리포지토리 생성
리포지토리는 비공개로 생성 (개인 계정, Organization 모두 가능)  

## 설정 파일 생성
생성한 비공개 리포지토리 내부에 Config Server가 읽어갈 설정 파일을 생성하자.  
이때 파일 명의 경우 아래와 같은 규칙이 필수적이다.  
`이름-환경.properties`, `이름-환경.yml`  
파일명은 대시(-)로 되어 있는 구분자를 필수로 넣어야 한다. 이름은 사용자가 식별할 수 있는 이름을 사용하고 환경은 dev, prod 와 같은 특정한 환경을 명시하면 된다.  

## 리포지토리 외부 접속을 위한 비대칭 키 생성
리포지토리를 외부에서 접속할 수 있도록 비대칭 키를 생성하며 public키를 깃허브 리포지토리에 등록해야 한다.  
이때 쉘 환경에서 ssh-keygen 명령어를 통해 비대칭키를 생성해야 한다.  

* 맥의 경우 터미널
* 윈도우의 경우 git bash

```
#!/bin/bash

ssh-keygen -m PEM -t rsa -b 4096 -C "코멘트(계정명 넣어도 됨)"
```

생성이 완료되면 /사용자/.ssh 경로에 생성 됨  
```
cd ~/.ssh
```
생성된 비대칭 키 중 public 키 내용을 복사하여 깃허브 리포지토리에 등록  

## Public 키 등록

생성한 리포지토리 - Settings - Depoly keys - Add Depoly key  

---
# 스프링에서 msa

## 참조
https://www.youtube.com/watch?v=VoonWkCJxcQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=2  

---
# 20240730
# 모니터링 메트릭 마무리, 실무팁, 마무리
## 메트릭등록 @Timed

어노테이션 활용하면 그냥 함수들 알아서 구해줌
```java
@Timed("my.order")
@Slf4j
public class OrderServiceV4 implements OrderService {
 private AtomicInteger stock = new AtomicInteger(100);
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 sleep(500);
 }
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 sleep(200);
 }
 private static void sleep(int l) {
 try {
 Thread.sleep(l + new Random().nextInt(200));
 } catch (InterruptedException e) {
 throw new RuntimeException(e);
 }
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
@Timed("my.order") 타입이나 메서드 중에 적용할 수 있다. 타입에 적용하면 해당 타입의 모든 public 메서드에 타이머가 적용된다.  

* config
```java
@Configuration
public class OrderConfigV4 {
 @Bean
 OrderService orderService() {
 return new OrderServiceV4();
 }
 @Bean
 public TimedAspect timedAspect(MeterRegistry registry) {
 return new TimedAspect(registry);
 }
}
```
TimedAspect 를 적용해야 @Timed 에 AOP가 적용된다.  

이렇게하면 엑츄에이터 프로메테우스 그라파나 모두 자동등록이되고 사용가능하다  

## 게이지 

오르고 내릴 수 있는 값들 

* 단순 등록 config에서 한번에
```java
@Configuration
public class StockConfigV1 {
 @Bean
 public MyStockMetric myStockMetric(OrderService orderService, MeterRegistry
registry) {
 return new MyStockMetric(orderService, registry);
 }
 @Slf4j
 static class MyStockMetric {
 private OrderService orderService;
 private MeterRegistry registry;
 public MyStockMetric(OrderService orderService, MeterRegistry registry)
{
 this.orderService = orderService;
 this.registry = registry;
 }
 @PostConstruct
 public void init() {
 Gauge.builder("my.stock", orderService, service -> {
 log.info("stock gauge call");
 return service.getStock().get();
 }).register(registry);
 }
 }
}
```
my.stock 이라는 이름으로 게이지를 등록했다.  
게이지를 만들 때 함수를 전달했는데, 이 함수는 외부에서 메트릭을 확인할 때 마다 호출된다. 이 함수의 반환 값이 게이지의 값이다.  

* 엑츄에이터 확인
```html

 "name": "my.stock",
 "measurements": [
 {
 "statistic": "VALUE",
 "value": 100
 }
 ],
 "availableTags": []
}
```
이런식으로 등록된다.  

* 좀더 단순하게 등록하기 config
```java
@Slf4j
@Configuration
public class StockConfigV2 {
 @Bean
 public MeterBinder stockSize(OrderService orderService) {
 return registry -> Gauge.builder("my.stock", orderService, service -> {
 log.info("stock gauge call");
 return service.getStock().get();
 }).register(registry);
 }
}
```

## 실무환경 구성 팁들
### 모니터링 3단계

* 대시보드
마이트로미터 프로메테우스 크라파나
전체적인 시각화, 시스템메트릭 cpu, 애플리케이션 메트릭 DB커넥션 호출수등, 비즈니스 메트릭 주문수, 취소수등

* 애플리케이션 추적
핀포인트(오픈소스), 스카우트(오픈소스), 와탭(상용), 제니퍼(상용)
핀포인트 추천

주로 각각의 HTTP 요청을 추적, 일부는 마이크로서비스 환경에서 분산 추적  

어떤 요청이 시간이 오래걸린지 확인하며 어떻게 msa서버들을 돌아다니고 sql이 어떻게 날라가고 한번에 확인가능  
문제해결에 가장 직접적인 도움  

* 로그
가장 자세한 추적 원하는데로 커스텀
다만 엄청난 로그들이 섞일건데 예전 강의에서 http요청마다 고유 id사용해서 묶을 수 있는 방법 미리 제공해주는것 있음 MDC 찾아서 적용해보기

파일로 로그 남길경우  
일반 로그와 에러 로그는 파일을 구분해서 남기자 에러 로그만 확인해서 문제를 바로 정리할 수 있음  

클라우드쓰면 검색 잘되도록 구분하기  

### 알람
모니터링 툴에서 일정 이상 수치가 넘어가면, 슬랙, 문자 등을 연동  
* 알람은 꼭 2가지로 구분해서 사용하자

경고, 심각  
경고는 하루 1번 정도 사람이 직접 확인해도 되는 수준(사람이 들어가서 확인)  
심각은 즉시 확인해야 함, 슬랙 알림(앱을 통해 알림을 받도록), 문자, 전화  

알람중에 이건 없애도 될거같은데.. 하는거 찾으면 바로바로 삭제해주자, 그러지않을경우 쓸모없는 알람이 쌓이면서 팀전체가 알림 확인 안하게되는 상황 생길 수 있음  

---
# 20240729
# MSA란

## 참고자료
https://www.youtube.com/watch?v=VoonWkCJxcQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=2

## MSA : Micro Service Architecture
MSA는 마이크로 서비스 아키텍처의 약자로 특정 서비스를 구축하는 방식을 의미한다.  

## 서비스 구축 방법
* 모놀로식
과거 부터 현재까지 주로 이용되는 서비스 구축 방식으로 하나의 프로젝트에 모든 비즈니스 로직과 설정 데이터들을 넣어 개발하는 방식이다

* MSA
모놀리식 서비스에서 각각의 비즈니스 로직을 분리하여 개별 프로젝트로 생성한 뒤 가장 앞단에서 API Gateway와 같은 분배기를 통해 각 서비스 서버에 요청을 분산하여 관리한다.

## 모놀리식과 MSA
![1](https://github.com/user-attachments/assets/90d1f340-ea58-4e3d-9946-4f75ddb4c181)  

## MSA 장단점
* 장점
서비스별 스케일링 가능  
서비스별 다른 프레임워크 사용 가능  
하나의 서비스가 off 되더라도 나머지 동작 가능  
부분적으로 로직 업데이트 가능

* 단점
초기 구성의 난이도  
시스템이 돌아가지만 특정 서비스가 off되면 재기능을 못할 수 있음  
서버간 호출 비용  
분산 관리

## MSA의 여러 요소

서비스 로직  
게이트웨이  
모니터링 서버  
변수관리 서버(여러 서버에서 DB관련정보 한번에 바꾸려면 변수주입서버 필요)  

## 스프링 프레임워크에서의 MSA
스프링 프레임워크도 MSA를 지원하기 위해 MSA 관련 의존성들을 제공한다.  

## 추가참고 영상
https://www.youtube.com/watch?v=CM47-1UpgOc  

---
# 20240716
# 메트릭등록 @Counted, Timer
## @Counted

이전의 방식은 비즈니스 로직 함수에 직접 코드를 쓰면서 유지보수가 힘들어진다. 근데 보면 AOP로 해결하면 딱 좋을거같은데  
역시 이 AOP를 미리 만들어준것이 있다 그걸 적용하면 된다.  

```java
package hello.order.v2;
import hello.order.OrderService;
import io.micrometer.core.annotation.Counted;
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.atomic.AtomicInteger;
@Slf4j
public class OrderServiceV2 implements OrderService {
 private AtomicInteger stock = new AtomicInteger(100);
 @Counted("my.order")
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 }
 @Counted("my.order")
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
함수명 위에 ` @Counted("my.order")`이것만 올려주면 클레스명, 함수명으로 각각의 태그도 만들어준다.  

```java
@Configuration
public class OrderConfigV2 {
 @Bean
 public OrderService orderService() {
 return new OrderServiceV2();
 }
 @Bean
 public CountedAspect countedAspect(MeterRegistry registry) {
 return new CountedAspect(registry);
 }
}
```
다만 aop클레스도 꼭 bean으로 추가 등록해줘야한다.  

```html
{
 "name": "my.order",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 5
 }
 ],
 "availableTags": [
 {
 "tag": "result",
 "values": [
 "success"
 ]
 },
 {
 "tag": "exception",
 "values": [
 "none"
 ]
 },
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v2.OrderServiceV2"
 ]
 }
 ]
}
```
액츄에이터에 보면 클레스명과 함수명으로 추가된것을 볼 수 있다.  

## Timer
timer는 시간을 측정하는데 사용된다.  
seconds_count : 누적 실행 수 - 카운터  
seconds_sum : 실행 시간의 합 - sum  
seconds_max : 최대 실행 시간(가장 오래걸린 실행 시간) - 게이지(내부에서 1~3분마다 새로운 최대값으로 갱신해줌)  

이 3가지를 한번에 해준다.  

```java
@Slf4j
public class OrderServiceV3 implements OrderService {
 private final MeterRegistry registry;
 private AtomicInteger stock = new AtomicInteger(100);
 public OrderServiceV3(MeterRegistry registry) {
 this.registry = registry;
 }
 @Override
 public void order() {
 Timer timer = Timer.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "order")
 .description("order")
 .register(registry);
 timer.record(() -> {
 log.info("주문");
 stock.decrementAndGet();
 sleep(500);
 });
 }
 @Override
 public void cancel() {
 Timer timer = Timer.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "cancel")
 .description("order")
 .register(registry);
 timer.record(() -> {
 log.info("취소");
 stock.incrementAndGet();
 sleep(200);
 });
 }
 private static void sleep(int l) {
 try {
 Thread.sleep(l + new Random().nextInt(200));
 } catch (InterruptedException e) {
 throw new RuntimeException(e);
 }
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
Timer.builder(name) 를 통해서 타이머를 생성한다. name 에는 메트릭 이름을 지정한다.  
tag 를 사용했는데, 프로메테우스에서 필터할 수 있는 레이블로 사용된다.  
주문과 취소는 메트릭 이름은 같고 tag 를 통해서 구분하도록 했다.  
register(registry) : 만든 타이머를 MeterRegistry 에 등록한다. 이렇게 등록해야 실제 동작한다.  
타이머를 사용할 때는 timer.record() 를 사용하면 된다. 그 안에 시간을 측정할 내용을 함수로 포함하면 된다.  

```java
@Configuration
public class OrderConfigV3 {
 @Bean
 OrderService orderService(MeterRegistry registry) {
 return new OrderServiceV3(registry);
 }
}
```
설정파일  

* 액츄에이터
```html
{
 "name": "my.order",
 "description": "order",
 "baseUnit": "seconds",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 5
 },
 {
 "statistic": "TOTAL_TIME",
 "value": 1.929075042
 },
 {
 "statistic": "MAX",
 "value": 0.509926375
 }
 ],
 "availableTags": [
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v3.OrderServiceV3"
 ]
 }
 ]
}
```
measurements 항목을 보면 COUNT , TOTAL_TIME , MAX 이렇게 총 3가지 측정 항목을 확인할 수 있다.  
COUNT : 누적 실행 수(카운터와 같다)  
TOTAL_TIME : 실행 시간의 합(각각의 실행 시간의 누적 합이다)  
MAX : 최대 실행 시간(가장 오래 걸린 실행시간이다)  

* 프로메테우스 포멧 메트릭 확인
```
# HELP my_order_seconds order
# TYPE my_order_seconds summary
my_order_seconds_count{class="hello.order.v3.OrderServiceV3",method="order",}
3.0
my_order_seconds_sum{class="hello.order.v3.OrderServiceV3",method="order",}
1.518434959
my_order_seconds_count{class="hello.order.v3.OrderServiceV3",method="cancel",}
2.0
my_order_seconds_sum{class="hello.order.v3.OrderServiceV3",method="cancel",}
0.410640083
# HELP my_order_seconds_max order
# TYPE my_order_seconds_max gauge
my_order_seconds_max{class="hello.order.v3.OrderServiceV3",method="order",}
0.509926375
my_order_seconds_max{class="hello.order.v3.OrderServiceV3",method="cancel",}
0.20532925
```
프로메테우스로 다음 접두사가 붙으면서 3가지 메트릭을 제공한다.  
seconds_count : 누적 실행 수  
seconds_sum : 실행 시간의 합  
seconds_max : 최대 실행 시간(가장 오래걸린 실행 시간), 프로메테우스 gague(1~3분마다 갱신)  

번외 평균시간 구하기 seconds_sum / seconds_count = 평균 실행시간  

* 그라파나 등록

주문수 V3  
increase(my_order_seconds_count{method="order"}[1m])  
increase(my_order_seconds_count{method="cancel"}[1m])  

최대시간  
my_order_seconds_max  

평균실행시간  
increase(my_order_seconds_sum[1m]) / increase(my_order_seconds_count[1m])  

---
## 20240626
### sql 5월 식품들의 총매출 조회하기 4단계

FOOD_PRODUCT와 FOOD_ORDER 테이블에서 생산일자가 2022년 5월인 식품들의 식품 ID, 식품 이름, 총매출을 조회하는 SQL문을 작성해주세요. 이때 결과는 총매출을 기준으로 내림차순 정렬해주시고 총매출이 같다면 식품 ID를 기준으로 오름차순 정렬해주세요.  

* FOOD_PRODUCT

|Column name|	Type|	Nullable|
|---|---|---
|PRODUCT_ID|	VARCHAR(10)|	FALSE|
|PRODUCT_NAME|	VARCHAR(50)|	FALSE|
|PRODUCT_CD|	VARCHAR(10)|	TRUE|
|CATEGORY|	VARCHAR(10)|	TRUE|
|PRICE|	NUMBER|	TRUE|

* FOOD_ORDER

|Column name|	Type|	Nullable|
|---|---|---|
|ORDER_ID|	VARCHAR(10)|	FALSE|
|PRODUCT_ID|	VARCHAR(5)|	FALSE|
|AMOUNT| NUMBER|	FALSE|
|PRODUCE_DATE|	DATE|	TRUE|
|IN_DATE|	DATE|	TRUE|
|OUT_DATE|	DATE|	TRUE|
|FACTORY_ID|	VARCHAR(10)|	FALSE|
|WAREHOUSE_ID|	VARCHAR(10)|	FALSE|

```
SELECT fp.PRODUCT_ID, fp.PRODUCT_NAME, fo.amount * fp.PRICE AS TOTAL_SALES
FROM FOOD_PRODUCT AS fp
LEFT JOIN (
    SELECT PRODUCT_ID, SUM(AMOUNT) AS amount 
    FROM FOOD_ORDER 
    WHERE DATE_FORMAT(PRODUCE_DATE, '%Y-%m') = '2022-05'
    GROUP BY PRODUCT_ID
) AS fo ON fp.PRODUCT_ID = fo.PRODUCT_ID
HAVING TOTAL_SALES IS NOT NULL
ORDER BY TOTAL_SALES DESC, fp.PRODUCT_ID;
```
감은 잡겠는데 암기해서 작성하지는 못하겠다... 상세한 문법이 조금 헷갈린다.

---
## 20240625
### 레디스 실사용

#### 레디스 설치
리눅스 우분투 22.04 환경  
```
#!/bin/bash

sudo apt update

sudo apt install redis-server
```

#### 레디스 설정

```
서버시작
sudo systemctl start redis-server

redis 서버 종료
sudo systemctl stop redis-server

redis 서버 상태 확인
sudo systemctl status redis-server

redis 서버 재시작
sudo systemctl restart redis-server

redis.conf 파일 위치
sudo vi /etc/redis/redis.conf

conf 파일 접근
sudo vi /etc/redis/redis.conf

비밀번호 설정
requirepass 비밀번호

redis-cli 비밀번호 입력
auth 비밀번호
//auth 사용자명 비밀번호

서버 재시작
sudo systemctl restart redis-server

Redis Bind 설정
conf 파일 접근
sudo vi /etc/redis/redis.conf

bind 항목 변경
bind 0.0.0.0 ::1

서버 재시작
sudo systemctl restart redis-server

스냅샷 설정
redis conf 파일 접속
sudo vi /etc/redis/redis.conf

save 파라미터 설정 : 스냅샷 주기
save 초단위시간 KEY변경수

save 900 1
save 300 10
save 60 10000

스냅샷 실패시 레디스 쓰기 요청 거부 설정
stop-wrotes-on-bgsave-error yes

스냅샷 압축 여부
rdbcompression yes

rdb 파일 마지막에 CRC64 checksum 기록 여부 (순환 중복 검사)
rdbchecksum yes

스냅샷 파일명 설정
dbfilename dump.rdb

스냅샷 파일 사용 후 제거 여부
rdb-del-sync-files no

스냅샷 저장 위치
dir /var/lib/redis
```

#### Redis CLI
```
Redis CLI 접근 : 로컬
redis-cli

Redis CLI 접근 : 원격
redis-cli -h IP주소 -p 포트 -a 비밀번호

Redis CLI 접근시 database 설정
redis-cli -n 데이터베이스번호

n 옵션을 통해 데이터베이스번호 명시가능, 이때 db번호는 0번부터 시작
```

#### Redis 데이터 타입

```
Redis 데이터 타입
레디스의 모든 데이터는 “key” - “value” 형태로 저장된다. 문자열 key에 대응되는 여러 타입의 value가 존재하며 그 종류는 아래와 같다.

String
단순 문자열 데이터로 최대 길이 512MB

Bitmaps
비트 단위의 이진수 데이터

Lists
배열형태의 데이터로 key와 value의 관계는 1 대 N이다. value의 최대 개수는 2^32 - 1 개며 주로 큐와 스택으로 활용된다.

Sets
집합 (중복되는 데이터가 없는 Lists)

Sorted Sets
집합에서 각각의 원소에 스코어라는 고유 숫자가 부여된다.

Hashes
json과 같은 데이터 형태로 key : value 내부에 key : value 가 존재한다.

HyperLogLogs
긴 길이의 Bitmaps 데이터

Streams
로그와 같은 스트림 데이터

//

데이터 CRUD
String 데이터

set : 데이터 쓰기
set KEY VALUE
//EX. set 1 kim

set과 함께 expire 시간 설정
set KEY VALUE EX 초단위시간
set KEY VALUE PX 밀리초단위시간

get : 데이터 읽기
get KEY
//EX. get 1

scan : 단위 키 조회
scan 단위숫자
//EX. scan 0

keys : 특정 키 조회
keys 패턴
//EX. keys *

exists : 키 존재 여부 확인
exists KEY
//EX. exists 1

del : 키 삭제
del KEY
//EX. del 1

flushall : 모든 키 삭제
flushall

//

### Lists 데이터


lpush : 좌측에서 데이터 추가
lpush KEY VALUE
//EX. lpush testlist a

rpush : 우측에서 데이터 추가
rpush KEY VALUE

lpop : 좌측에서 데이터 제거
lpop KEY
//EX. lpop testlist

rpop : 우측에서 데이터 제거
rpop KEY

llen : 저장된 데이터 개수 출력
llen KEY

lrange : 지정된 단위 VALUE 출력
lrange KEY 시작인덱스 종료인덱스
```

#### Replication, 클러스터

### Replication : 복제

Replication은 Redis에서 지원하는 복제 로직으로 동일한 데이터에 대해서 마스터 - 슬레이브 구성을 갖추어 마스터 서버에 장애가 발생하는 경우 슬레이브 서버를 통해 마스터를 대체할 수 있도록 구성할 수 있다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/1c76df36-ccc5-42fb-931c-05a506ffb13d)  

Replication 설정  

복제의 경우 슬레이브 노드의 conf 파일에서 설정을 진행한다  

```
redis.conf 파일
sudo vi /etc/redis/redis.conf

복제 설정
replicaof 마스터IP 마스터포트
//EX. replicaof 127.0.0.1 6379

설정 후 재시작
sudo systemctl restart redis-server
```

마스터 서버 장애 대비  
마스터 서버에 장애가 발생하였을 경우 슬레이브 서버를 마스터 서버로 승격시켜야 한다.  
이때 수동으로 설정을 진행하는 것이 거의 불가능하기 때문에 Redis는 Sentinel이라는 도구를 지원한다.  

* 클러스터 사용 이유

물리적인 휘발성 메모리의 저장용량 한계로 인해 한계 이상의 데이터를 입력해야하는 경우 여러개의 노드를 클러스터링시켜 저장공간을 확보시킬 수 있다.  
단일 노드 구성대비 서버 장애에 대한 부분 동작이 가능하다.  

클러스터 : 마스터 - 슬레이브  
클러스터 설정시 마스터 - 슬레이브 설정 또한 진행되면 장애 발생시 자동으로 슬레이브 → 마스터 승격 또한 가능하다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/a43309c3-0ca6-47b0-8ef9-726e21ecaf73)  

클러스터 조건  

1. 레디스 클러스터는 최소 3개 이상의 노드로 구성된다.  
2. 클러스터 기능을 사용하면 database 기능을 활용하지 못하고 database는 0번 인덱스만 사용가능하다.  
3. 클러스터간 통신을 위해 기본포트 + 10000번 포트를 오픈 (16379)

클러스터 설정  

```
redis.conf 파일 접근
sudo vi /etc/redis/redis.conf

protected-mode 종료
protected-mode no

클러스터 설정
cluster-enavled yes

클러스터 구성 파일 이름 설정 (자동 생성)
cluster-config-gile nodes-6379.conf

노드간 통신 장애시 타임아웃 설정 (10000 = 10s)
cluster-node-timeout 10000

appendonly 실행
appendonly yes

재시작
sudo systemctl restart redis-server

redis-cli로 클러스터 시작 명령
redis-cli --cluster create 아이피:포트 아이피:포트 아이피:포트

추가로 클러스터 생성시 슬레이브 노드 옵션을 추가하려면
—cluster-replicas 개수
```

#### AWS ElastiCache

AWS ElastiCace는 AWS에서 제공하는 SaaS형 레디스로 성능, 리전, 클러스터 설정만 GUI로 진행하면 추후 발생하는 모든 유지보수(노드 추가, 장애 대응, 업데이트)는 AWS에서 관리하게 된다.  
ElastiCache의 경우 Redis와 Memcached 엔진을 지원한다.  

```
ElastiCache : Redis 생성
AWS 콘솔 홈 접근 후 ElastiCache 접속
Redis 클러스터 클릭
Redis 클러스터 생성 클릭
새 클러스터 구성 및 생성 선택 후

클러스터 모드를 설정할 수 있다. 
클러스터 모드 활성화시 다중 노드를 통해 스케일 아웃 및 Master-Slave 구조를 형성할 수 있으며 비활성화시 단일 마스터 노드로 구성된다.

클러스터 이름 설정
클러스터 생성 위치 설정

기본적으로 AWS 클라우드이며 온프레미스의 경우 실물 서버를 소유하고 있는 경우 연결할 수 있다.
다중 AZ는 클러스터 모드 활성화시 여러 AZ에 걸쳐 ElastiCache 노드를 분산하여 고가용성을 유지할 수 있다.

Redis 버전, 포트, 파라미터값, 노드 성능, 샤드 개수, Slave 노드 수 설정
서브넷 설정

클러스터 데이터 입력시 노드별 균등 저장 방법
암호화 및 보안그룹 설정

보안 그룹 설정을 통해 클러스터에 접근할 포트 오픈을 진행해야 한다.
(추후 보안 그룹에 대한 설정은 : 콘솔홈 - EC2 - 보안그룹)

백업 관련 설정

업데이트 및 로그 관련 설정
이후 확인사항을 거쳐 생성을 할 수 있다.

//


접속 원리

ElastiCache는 외부 접속이 불가능하며 AWS 내부 접속만 가능하다. 따라서 AWS EC2를 사용하여 접속을 진행한다.

AWS EC2 생성
EC2에 레디스 서버 설치
ElastiCache가 존재하는 보안그룹 6379 포트 열기
EC2에서 redis-cli로 원격 접속 진행

클러스터 IP는 ElastiCache 대시보드에서 확인할 수 있다.

기본 엔드포인트 참조 (IP:포트)

//

접속

보안그룹 설정
ElastiCache가 존재하는 보안 그룹을 선택하여 6379 포트 오픈

AWS EC2 접속
레디스 서버가 설치된 EC2 접속 후 아래 명령어로 접근
redis-cli -h 아이피주소 -p 6379

IP의 경우 기본엔드포인트 참조
```


---
## 20240624
### 메트릭 등록 카운터 기본

비즈니스로직적인 부분을 직접 메트릭등록을해서 그라파나에서 확인할 수 있도록 만들어보자  

주문을 하면 개수가올라가고, 최소를 하면 개수가 줄어들되
메트릭에는 주문을 하면 1씩 증가 취소를해도 취소요청개수가 1씩증가 이런식으로 만들어보자

* controller
```java
@Slf4j
@RestController
public class OrderController {
 private final OrderService orderService;
 public OrderController(OrderService orderService) {
 this.orderService = orderService;
 }
 @GetMapping("/order")
 public String order() {
 log.info("order");
 orderService.order();
 return "order";
 }
 @GetMapping("/cancel")
 public String cancel() {
 log.info("cancel");
 orderService.cancel();
 return "cancel";
 }
 @GetMapping("/stock")
 public int stock() {
 log.info("stock");
 return orderService.getStock().get();
 }
}
```

* service
```java
package hello.order.v1;
import hello.order.OrderService;
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.atomic.AtomicInteger;
@Slf4j
public class OrderServiceV1 implements OrderService {
 private final MeterRegistry registry;
 private AtomicInteger stock = new AtomicInteger(100);
 public OrderServiceV1(MeterRegistry registry) {
 this.registry = registry;
 }
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 Counter.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "order")
 .description("order")
 .register(registry).increment();
 }
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 Counter.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "cancel")
 .description("order")
 .register(registry).increment();
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
기본적으로 `private final MeterRegistry registry;`로 등록이 가능하다. static 빌더로 이름, tag들, `.register(registry).increment();`사용되면 증가
이런식으로 등록하면된다.  

* config
```java
package hello.order.v1;
import hello.order.OrderService;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class OrderConfigV1 {
 @Bean
 OrderService orderService(MeterRegistry registry) {
 return new OrderServiceV1(registry);
 }
}
```
config 등록

각 엔드포인드를 요청후에 

* http://localhost:8080/actuator/metrics/my.order
```html
{
 "name": "my.order",
 "description": "order",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 2
 }
 ],
 "availableTags": [
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v1.OrderServiceV1"
 ]
 }
 ]
}
```
들어가보면 기본적으로 엑츄에이터에도 등록되어있고

* http://localhost:8080/actuator/prometheus
```html
# HELP my_order_total order
# TYPE my_order_total counter
my_order_total{class="hello.order.v1.OrderServiceV1",method="order",} 1.0
my_order_total{class="hello.order.v1.OrderServiceV1",method="cancel",} 1.0
```
프로메테우스에도 자기 형식에 맞게 조금 바뀌어서 등록되어진다.  

이걸기반으로 그라파나에서도 요청해서 사용할 수 있다.  

* increase(my_order_total{method="order"}[1m])
* Legend : {{method}}
* increase(my_order_total{method="cancel"}[1m])
* Legend : {{method}}


---
## 20240621
### sql 주문량이 많은 아이스크림 조회하기 4단계

* FIRST_HALF 
|NAME|TYPE|NULLABLE|
|---|---|---|
|SHIPMENT_ID(외래키)|INT(N)|FALSE|
|FLAVOR(기본키)|VARCHAR(N)|FALSE|
|TOTAL_ORDER|INT(N)|FALSE|

* JULY
|NAME|TYPE|NULLABLE|
|---|---|---|
|SHIPMENT_ID(기본키)|INT(N)|FALSE|
|FLAVOR(외래키)|VARCHAR(N)|FALSE|
|TOTAL_ORDER|INT(N)|FALSE|

7월 아이스크림 총 주문량과 상반기의 아이스크림 총 주문량을 더한 값이 큰 순서대로 상위 3개의 맛을 조회하는 SQL 문을 작성해주세요.  

즉 JULY에서는 같은 FLAVOR에 다른 SHIPMENT_ID로 TOTAL_ORDER여러개 있을 수 있어서 1차적으로 합치고
그다음 TOTAL_ORDER에서 TOTAL_ORDER한번더 합쳐줘야함  

```sql
SELECT fh.FLAVOR
FROM FIRST_HALF AS fh
LEFT JOIN (
    SELECT flavor, SUM(total_order) AS total_order
    FROM JULY
    GROUP BY flavor
) AS jy ON fh.flavor = jy.flavor
ORDER BY fh.total_order + jy.total_order DESC
LIMIT 3;
```
서브쿼리 이용 문제였음 위와같은 상황일경우 서브쿼리로 1차적으로 한번 합치고 그다음 다시 더하는 연산 해야하는듯  
