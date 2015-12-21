# How to apply Spring Security

>Spring Security carries out a full framework to apply security functionality to your application. In this case , we apply it to RESTful api, as well as AngularJS frontend.

Spring Security provides security functionality throuth authentication and authorization. 

##Authentication
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

There are three roles in operation management system, they are "admin", "manager" and "marketing"

```java
public class User {
    public User() {
        
        int nIndex = 10;
        
        System.out.println(nIndex);
    }
}
```

This is code block.
