# Usage #

The project consists of a [Spring Security Filter](http://static.springsource.org/spring-security/site/docs/3.0.x/reference/springsecurity-single.html#ns-custom-filters) which is protected by a [spring oauth protected resource](http://static.springsource.org/spring-security/oauth/oauth2.html#Managing_Protected_Resources).

The filter is inserted after the spring oauth filter chain and can therefore only be accessed after the user has logged into facebook and granted access using OAuth2. The filter will then forward the authentication request to the Facebook authentication provider, which will retrieve the user details using the [facebook Graph API](http://developers.facebook.com/docs/reference/api/) and will then authorise against your application using a provided [UserDetailsService](http://static.springsource.org/spring-security/site/docs/3.0.x/apidocs/org/springframework/security/core/userdetails/UserDetailsService.html).

In order to use the authentication filter, you have to display the following form on your login page (you can hide it and submit it using javascript):
```
<form action="/newapp/j_spring_oauth_security_check" method="POST" enctype="application/x-www-form-urlencoded">
	<input type="submit">
</form>
```
This will then forward the user to facebook for the OAuth2 authorization and will then redirect to the page the user came from. Make sure you adapt the action URL to suit your setup.

In order to get a valid setup, you have to add the following snippet to your spring configuration and include `classpath:/config.spring-security-social.xml` in your spring configuration. Here is the snippet:

```
	<!-- Spring security configuration -->
	<bean id="userDetailsService" class="com.example.newapp.SecurityService"/>
	<sec:authentication-manager alias="authenticationManager">
		<sec:authentication-provider user-service-ref='userDetailsService'/>
		<sec:authentication-provider ref="facebookAuthenticationProvider"/>
	</sec:authentication-manager>
	<bean id="oauth2ResourceDetailsService" class="org.springframework.security.oauth2.config.ResourceDetailsServiceFactoryBean"/>
	<sec:http>
 		<sec:custom-filter before="FILTER_SECURITY_INTERCEPTOR " ref="oauth2AuthFilter"/>
		<sec:logout />
	</sec:http>
	<!-- You can exchange that with your custom token service -->
	<bean id="oauth2TokenServices" class="org.springframework.security.oauth2.consumer.token.InMemoryOAuth2ClientTokenServices" />
	<!--define an oauth 2 resource for facebook. according to the facebook docs, the 'clientId' 
		is the App ID, and the 'clientSecret' is the App Secret -->
	<oauth:resource id="facebook" type="authorization_code"
		clientId="202086623166314" clientSecret="0838b1813f49ad5625e7c518c0741439"
		bearerTokenMethod="query" accessTokenUri="https://graph.facebook.com/oauth/access_token"
		scope="email" userAuthorizationUri="https://www.facebook.com/dialog/oauth" />
```

Things you need to change are:
  * **userDetailsService**: This should be a service implementing UserDetailsService, which maps from username strings like **oauth2:facebook:XXX** to UserDetails objects.
  * **clientId="xxx" clientSecret="xxx"**: This will be you facebook App ID and secret, which you have created using the [facebook developers app](http://www.facebook.com/developers/).

Things you can change:
  * **sec:http**: You might want to add alternative authorisation methods such as openid here
  * **scope="email"**: You might request additional [facebook permissions](http://developers.facebook.com/docs/authentication/permissions/).
  * **oauth2TokenServices**: You can use a different implementation which keeps oauth tokens between restarts if you like.

**ATTENTION**: Make sure you add all the domains you use to redirect to in your facebook app settings. See [stackoverflow](http://stackoverflow.com/questions/7382698) and [developers.facebook.com](https://developers.facebook.com/blog/post/570/) for more details.

One problem you still have to solve is that you have to create users automatically in the background as soon as the filter has granted access. Here is some pseudocode:

```
public class SecurityService implements UserDetailsService, ApplicationListener<InteractiveAuthenticationSuccessEvent>{
	public static class CustomUserDetails implements UserDetails {
		private static final long serialVersionUID = -7329960048878759841L;
		String userName;
		...
		Collection<GrantedAuthority> fakeAuthorities = new LinkedList<GrantedAuthority>();
		
		public CustomUserDetails(String userName, String password) {
			...
			fakeAuthorities.add(new GrantedAuthorityImpl("ROLE_USER"));
			fakeAuthorities.add(new GrantedAuthorityImpl("ROLE_ADMIN"));
		}
		... getters / setter ...
	}
	public UserDetails loadUserByUsername(String username)
			throws UsernameNotFoundException, DataAccessException {
		... Fetch existing, or create new user object in the database ...
		return new CustomUserDetails(username, "RANDOMPASSWORD");
	}
	/** This method is called whenever authentication with a third party
	 * authentication provider (such as OpenID and Facebook) succeeded. We
	 * copy user information from the authentication provider into our system.
	 */
	@Override
	public void onApplicationEvent(InteractiveAuthenticationSuccessEvent event) {
		Authentication auth = event.getAuthentication();
		if(auth instanceof OAuth2AuthenticationToken) {
			OAuth2AuthenticationToken oAuth2 = (OAuth2AuthenticationToken)auth;
			if(oAuth2.getProviderAccountDetails() instanceof com.restfb.types.User) {
				// This is a facebook user
				com.restfb.types.User fbUser = (com.restfb.types.User)oAuth2.getProviderAccountDetails();
				CustomUserDetails userDetails = (CustomUserDetails)SecurityContextHolder.getContext().getAuthentication().getPrincipal();
				updateDbUser(userDetails, fbUser.getFirstName(), fbUser.getLastName(), null);
			}
		}
	}
	
	void updateDbUser(CustomUserDetails userDetails, String forename, String surname, String email) {
		... Update userDetails with new values ...
	}
}
```