domain.xml 혹은 standalone.xml에 아래와 같은 내용을 넣는다

character encoding filter에서 보톤 utf8로 넣어주면 해결되는 사례가 많으나
이전 버전의 WAS, J2EE 컨테이너 등의 경우엔 아래와 같은 설정이 기본으로 없는 경우도 많음...

```xml
<system-properties>
        <property name="java.net.preferIPv4Stack" value="true"/>
        <property name="org.apache.catalina.connector.URI_ENCODING" value="UTF-8"/>
        <property name="org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING" value="true"/>
        <property name="org.apache.tomcat.util.http.Parameters.MAX_COUNT" value="10000"/>
        <property name="jboss.modules.system.pkgs" value="org.jboss.logmanager,com.navercorp.pinpoint.bootstrap,com.navercorp.pinpoint.common,com.navercorp.pinpoint.exception" boot-time="true"/>
        <property name="java.util.logging.manager" value="org.jboss.logmanager.LogManager" boot-time="false"/>
    </system-properties>
    
```
