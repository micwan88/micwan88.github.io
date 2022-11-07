---
title: Handle CORS preflight request in Spring security
categories: java spring cors
---

According to [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests), browser will trigger CORS preflight (e.g. HTTP's OPTIONS request) when the request is not a CORS simple request (i.e. not HTTP's GET, HEAD, POST). So sometimes we may encounter CORS error / HTTP status 403 error when we testing our front-end with HTTP PUT or DELETE call when cross-origin.

In fact, you can solve the problem by either way as below. (However, please ensure this is for testing only as it may cause security issue if it is for production).

- Disable CORS checking in the browser like Chrome
- Enable CORS in your web server like apache or nginx
- Enable CORS in backend "e.g. your front-end origin (e.g. `localhost:4200`) access to the backend (e.g. `localhost:8080`)"

## Disable CORS checking in the browser like Chrome or Chromium

We can same origin policy in Chrome / Chromium by adding command line option when start it up. After that, browser will not block your CORS traffic.

``` bash
#chromium
chromium-browser --disable-web-security --user-data-dir=/the-path-for-chrome-to-create-temp-user-directory

#chrome
chrome --disable-web-security --user-data-dir=/the-path-for-chrome-to-create-temp-user-directory
```

For Firefox, the settings are in `about:config` -> `security.fileuri.strict_origin_policy` -> `false`.
However, I have tried and I don't know why it only work for simple request (e.g. GET) and it didn't work for PUT and DELETE.
According to stackoverflow, it may require to install a plugin and I didn't try. Please let me know if someone tried and working fine.

## Enable CORS in your web server like apache or nginx

Add below headers to the response of your web server, so that it appear in every response from your web server.

```
Access-Control-Allow-Origin: http://localhost:4200
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, HEAD, OPTIONS
Access-Control-Allow-Headers: *

##This is required for browser to send your request with credentials like cookies with session ID
Access-Control-Allow-Credentials: true
```

Also, if you have specical header for your backend like CSRF token, then you have to expose it by using below header

```
Access-Control-Expose-Headers: xxxx1, xxxx2
```

## Enable CORS in backend

In Spring, we can enable CORS handling by define either `CorsConfigurationSource` or `WebMvcConfigurer`.

- For `CorsConfigurationSource`

``` java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http.cors().and()
			... //continue to do your setting ...
		return http.build();
	}

	@Bean
	CorsConfigurationSource corsConfigurationSource() {
		CorsConfiguration configuration = new CorsConfiguration();
		//Allow for localhost with all ports
		configuration.setAllowedOriginPatterns(Arrays.asList("http://localhost:[*]"));
		//Include preflight request of OPTIONS
		configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "HEAD", "OPTIONS"));
		//I found that below one is required for preflight request, because browser need "Access-Control-Allow-XXXX" headers
		//According to MDN Web Docs, the CORS safelisted header are only Accept, Content-type, ... etc 
		configuration.setAllowedHeaders(Arrays.asList("*"));
		//If you have specical header for your backend like CSRF token, then you have to expose it all
		configuration.setExposedHeaders(Arrays.asList("xxxx1", "xxxx2"));
		//This is required for browser to send your request with credentials like cookies with session ID
		//Please noted that it is required to enable for sending the credentials in front-end too (e.g. in Angular HttpClient)
		configuration.setAllowCredentials(true);
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", configuration);
		return source;
	}
}
```

- For `WebMvcConfigurer`

``` java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http.cors().and()
			... //continue to do your setting ...
		return http.build();
	}

	@Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
				.allowedOriginPatterns("http://localhost:[*]") //Allow for localhost with all ports
				.allowedMethods("GET", "POST", "PUT", "DELETE", "HEAD", "OPTIONS") //Include preflight request of OPTIONS
				.allowedHeaders("*") //I found that below one is required for preflight request, because browser need "Access-Control-Allow-XXXX" headers
				.exposedHeaders("xxxx1", "xxxx2") //If you have specical header for your backend like CSRF token, then you have to expose it all
				.allowCredentials(true); //This is required for browser to send your request with credentials like cookies with session ID
            }
        };
    }
}
```

References:
- [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Spring Security - CORS Support](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)
- [Spring CORS](https://www.baeldung.com/spring-cors)
- [Spring security CORS preflight](https://www.baeldung.com/spring-security-cors-preflight)
- [Disable same origin policy in Chrome](https://stackoverflow.com/questions/3102819/disable-same-origin-policy-in-chrome)
- [Disable same origin policy in Firefox](https://stackoverflow.com/questions/17088609/disable-firefox-same-origin-policy)