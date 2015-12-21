# How to apply Spring Security

>Spring Security carries out a full framework to apply security functionality to your application. In this case , we apply it to RESTful api, as well as AngularJS frontend.

Spring Security provides security functionality throuth authentication and authorization. 

##Authentication

###User Authentication
Authentication is to verify a user can access to the system. In this case, we use a customized ``UserDetailService`` named ``OpsUserDetailService`` to authenticate a user, the code as the following:

```java
package ms.zui.operation.security;

import java.util.ArrayList;
import java.util.List;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

import ms.zui.operation.Application;
import ms.zui.operation.datamodel.domain.User;


@Component
public class OpsUserDetailService implements UserDetailsService {

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		
		User user = Application.userService.getUserByName(username);
		
		if(user == null) {
			throw new UsernameNotFoundException(String.format("User with the username %s doesn't exist", username));
		}
		
		// Create a granted authority based on user's role. 
		// Can't pass null authorities to user. Hence initialize with an empty arraylist
		List<GrantedAuthority> authorities = new ArrayList<>();
		
		switch (user.getRole()) {
			
		case "admin": 
			authorities = AuthorityUtils.createAuthorityList("ROLE_ADMIN");
			break;
		case "manager":
			authorities = AuthorityUtils.createAuthorityList("ROLE_MANAGER");
			break;
		case "marketing":
			authorities = AuthorityUtils.createAuthorityList("ROLE_MARKETING");
			break;
		default:
			authorities = AuthorityUtils.createAuthorityList("ROLE_MARKETING");
		}
		
		// Create a UserDetails object from the data 
		UserDetails userDetails = new org.springframework.security.core.userdetails.User(user.getName(), user.getPassword(), authorities);
		
		return userDetails;
	}
}
```

There are three roles in operation management system, they are "admin", "manager" and "marketing" and mapped to "ROLE_ADMIN", "ROLE_MANAGER" and "ROLE_MARKETING" in function ``loadUserDetailByName`` respectively. And then, we should implement a function in ``classs SecurityConfiguration`` in order to make ``OpsUserdetailService`` effect. 

###Resources Authentication
Now, the user authentication is completed. Besides it, we still have to config the authentication of  the access to resouces, such as a URL, RESTful API. ``protected void configure(HttpSecurity http)`` in ``class SecurityConfiguration`` is the place to configure resouces authentication.

```java
package ms.zui.operation.security;

import javax.inject.Inject;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.security.SecurityProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableRedisHttpSession
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Inject
	private OpsUserDetailService opsUserDetailService;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();
		http
			.authorizeRequests()
				.regexMatchers(HttpMethod.OPTIONS, "/users", "/users/logout").permitAll()
				.regexMatchers(HttpMethod.OPTIONS, "/restaurants").permitAll()
				.antMatchers("/token").permitAll()
				.anyRequest().authenticated();
		
		http.csrf().disable();
	}
	
	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		auth
			.userDetailsService(opsUserDetailService);
				//.passwordEncoder(new BCryptPasswordEncoder());imp
	}
}
```
``@EnableRedisHttpSession`` indicates that using http session stored in Redis to authenticate a request.

``@EnableGlobalMethodSecurity(prePostEnabled = true)`` will be discussed in next section.

