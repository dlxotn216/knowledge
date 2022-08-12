# Spring Security로 인해 특정 시간이 지난 후 Cache 헤더가 응답되지 않는 현상


## 문제 현상
* 특정 프로젝트에 적용된 Resource Cache를 위한 Cache-Control 헤더가 일정 시간이 지남에 따라 변경되어 브라우저 캐시가 적용되지 않는 현상  
* 다른 프로젝트에선 잘 동작하고 있기에 원인 파악이 늦어저 이관이 지연됨  
* CDN의 Cache revalidate 후 정상적으로 헤더가 내려오나 일정 시간이 지나고 다시 문제가 발생  

캐시를 적용한 응답이 왔으나 일정 시간 후 아래의 캐시 무효화 헤더가 붙음  
Cache-Control: no-cache, no-store, max-age=-, must-revalidate  

### (1) Spring MVC 설정 확인
```xml
<mvc:resources mapping="/resources/**" location="/resources/" />
<mvc:resources mapping="/resources/css/**" location="/resources/css/" cache-period="21600"/>
<mvc:resources mapping="/resources/image/**" location="/resources/image/" cache-period="21600"/>
<mvc:resources mapping="/resources/fonts/**" location="/resources/fonts/" cache-period="2592000"/>
<mvc:resources mapping="/resources/javascript/library/**" location="/resources/javascript/library/" cache-period="2592000"/>
<mvc:resources mapping="/resources/javascript/**" location="/resources/javascript/" cache-period="0" />
```
스프링의 리소스 캐시 설정이 적절히 걸려 있었음  

### (2) 별도로 위와 같은 캐시 헤더를 추가하는 필터 등이 프로젝트내에 존재하지 않는 것을 확인 

### (3) Spring Security에서 Default Security Headers를 추가하는 데 위에 기술 된 헤더들이 추가 됨.  
https://docs.spring.io/spring-security/site/docs/5.0.x/reference/html/headers.html 

해당 설정을 비활성화 하는 방법으로 아래와 가이드를 찾고 XML에서 아래 Java config를 임포트 하도록 권장하였으나 설정이 양쪽으로 나뉘는 것이 불편하다고 하여 다른 방법 필요.  
```java
@Configuration
@EnableWebMvcSecurity
class SpringWebSecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        http.headers().cacheControl().disable();
    }
}
```

### (4) xml 기반의 설정을 찾고 추가 
Security의 문제가 맞는 지 확인하기 위해 모든 캐시 관련 설정을 끄고 확인
```xml
<http>
   ...
  <headers disabled="true" />
</http>

<!-- 혹은 아래와 같이 cacne control 헤더에 대한 것만 비활성화 처리 -->
<headers defaults-disabled="true">
    <cache-control/>
</headers>
```

시간이 지나도 정상적으로 캐시가 유지되는것을 확인.  

### (5) 다른 프로젝트의 security 설정 비교하여 원인 파악
다른 프로젝트는 위와 같은 설정이 없음에도 잘 동작 했기에 security 설정을 위주로 비교  

아래와 같이 리소스에 대해서는 security를 none으로 설정해서 SecurityFilter를 아예 안타게 함
```xml
<security:http pattern="/resources/**" security="none" />
<security:http pattern="/css/**" security="none" />
<security:http pattern="/images/**" security="none" />
<security:http pattern="/js/**" security="none" />
```

문제가 발생하는 프로젝트에선 아래와 같이 리소스에 대해서는 permitAll을 해서 SecurityFilter가 타게 함.  
-> 여기서 Cache-Control을 무효화 하는 헤더가 붙어버리는 것으로 파악  
```xml
<security:intercept-url pattern="/resources/**" access="permitAll"/>
```

### (6) 최종 설정을 반영해서 배포 완료 및 의문점 FU
왜 일정시간이 지나고서야 Cache-Control: no-cache, no-store, max-age=-, must-revalidate 헤더가 응답되어 캐시가 안되었을까?  

* 최초에는 CDN이 revalidate 되었으니 Origin에 접속하여 Spring MVC에서 설정한 cache-period에 의해 적절히 응답 헤더가 구성 됐을 것  
* max-age가 지나고 나서 각각의 resource에 대해 origin에 요청을 날렷는데 호출체인의 끝에 걸린 리소스가 SecurityFilter에 걸린 경우 모든 리소스가 그 헤더의 응답을 따랐갔을 가능성이 있음

https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Expiration.html


