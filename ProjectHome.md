# About #
This extension provides an NTLM authentication via Jespa library to Spring Framework.
# Usage #
## Step 1 - configure AD account ##
Visit a webpage o Jespa project and read operator's manual for current version (obsolete). Configure an Active Directory machine account using _SetupWizard_ vbs script. Generated config file copy to your WEB-INF directory using name _ntlm.ptp_.
## Step 2 - attach Ntlm filter ##
In your web.xml configure additional filter and map to certain path:

```
<filter>
   <filter-name>jespaFilter</filter-name>
   <filter-class>jespa.http.HttpSecurityFilter</filter-class>
   <init-param>
      <param-name>properties.path</param-name>
      <param-value>WEB-INF/ntlm.prp</param-value>
   </init-param>
</filter>

<filter-mapping>
   <filter-name>jespaFilter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

## Step 3 - configure application context ##
At your web application context configure following beans:

```
<bean id="filterChainProxy" class="org.springframework.security.web.FilterChainProxy">
   <security:filter-chain-map path-type="ant">
   <security:filter-chain pattern="/**"
     filters="securityContextPersistenceFilter,exceptionTranslationFilter,ntlmFilter,filterSecurityInterceptor" />
   </security:filter-chain-map>
</bean>

<bean id="securityContextPersistenceFilter" class="org.springframework.security.web.context.SecurityContextPersistenceFilter">
   <property name='securityContextRepository'>
      <bean class='org.springframework.security.web.context.HttpSessionSecurityContextRepository'		 p:allowSessionCreation="false" />
   </property>
</bean>

<bean id="accessDeniedHandler" class="org.springframework.security.web.access.AccessDeniedHandlerImpl" />

<bean id="exceptionTranslationFilter" class="org.springframework.security.web.access.ExceptionTranslationFilter">
   <property name="authenticationEntryPoint" ref="ntlmEntryPoint" />
   <property name="accessDeniedHandler" ref="accessDeniedHandler" />
</bean>


<bean id="ntlmEntryPoint" class="org.springframework.security.jespa.authentication.NtlmAuthenticationEntryPoint" />

<bean id="ntlmFilter" class="org.springframework.security.jespa.filter.NtlmSecurityFilter" p:authenticationEntryPoint-ref="ntlmEntryPoint" p:authenticationManager-ref="providerManager" />

<bean id="providerManager" class="org.springframework.security.authentication.ProviderManager">
   <property name="providers">
      <list>
         <ref bean="ntlmAuthenticationProvider" />
      </list>
   </property>
</bean>

<bean id="ntlmAuthenticationProvider" class="org.springframework.security.jespa.provider.NtlmAuthenticationProvider" p:userDetailsService-ref="userDetailsService" />
```

All you need is additional bean named userDetailsService (your own implementation which loads users (with credentials) by name from db or f.e. property file). For details see: [UserDetailsService interface](http://static.springsource.org/spring-security/site/docs/3.0.x/apidocs/org/springframework/security/core/userdetails/UserDetailsService.html). You need an accessDecisionManager and filterSecurityInterceptor but for details read Spring Security tutorial. Important: configuration sample is Acegi style but it's not a hint / problem.


# Links #
  * [Spring Framework](http://www.springsource.org)
  * [Jespa](http://www.ioplex.com/jespa.html)
  * [Spring Security module](http://static.springsource.org/spring-security/site/features.html)
