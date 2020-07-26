# Spring Security

## 常用拦截器

SecurityContextPersistenceFilter

- 以前是HttpSesstionContextIntegrationFilter,位于过滤器的顶端，是第一个起作用的过滤器
- 第一个用途：在执行其他过滤器之前，率先判断用户的session是否已经存在了一个spring security上下文的securityContext,如果存在，就把securityContext拿出来，放在securityContextHolder中，供security的其他部分使用。如果不存在，就创建一个securityContext出来，放在securityContextHolder中，供security的其他部分使用。
- 第二个用途：在所有过滤器执行完毕后，清空securityContextHolder中的内容，因为securityContextHolder是基于ThreadLocal的，如果不清空，会受到服务器线程池机制的影响。
- ThreadLocal存放的值是线程内共享的，线程间互斥的，主要用于线程内共享一些数据，避免通过参数来传递。这样处理后，能够解决实际中的一些并发问题。ThreadLocalMap是ThreadLocal的一个内部类，是不对外使用的。当使用ThreadLocal存值时，首先获取到当前线程对象，然后获取到当前线程本地对象，本地变量map,最后将当前使用的所有local和传入的值放在map中。也就是说ThreadLocalMap中的key是ThreadLocal对象。这样，每个线程都对应一个本地的map,所以，一个线程可以存在多个线程本地变量。
- ThreadLocal是解决线程并发问题的一个很好的思路，通过对每个线程提供一个独立的变量副本，解决线程并发访问变量的一个冲突问题
- 当一个线程结束的时候，记得把ThreadLocal里的变量移除掉remove();



LogoutFilter

> 只处理注销请求。在用户发送注销请求时，销毁用户的session,清空securityContextHolder,重定向到注销成功页面



AbstractAuthenticationProcessingFilter

- 处理form登录的过滤器，与form登录有关的操作都在此进行。



DefaultLoginPageGeneratingFilter

- 用来生成一个默认的登录页面，默认的访问地址为spring_security_login,这个登录页面虽然支持用户输入用户名密码，也支持remember me等功能，但是因为太难看了，只能在演示时做个样子，不能直接在实际项目中使用



BasicAutenticationFilter

- 用来进行basic验证



SecurityContextHolderAwareRequestFilter

- 用来包装客户的请求，目的是在原来请求的基础上，为后续程序提供一些额外的数据，比如getRomoteUser时，直接返回当前登录的用户名



RememberMeAuthenticationFilter

- 实现Remember me功能，当用户cookie中存在remember me标记时，它会根据标记自动实现用户登录，并创建securityContext,授予对应的权限。spring security中的remember me依赖cookie实现，用户在登录时选择remember me,系统就会在登录成功后为用户生成一个唯一的标识，并将这个标识保存进cookie中，我们可以通过浏览器查看用户电脑中的cookie



AnonymousAutenticationFilter

- 当用户没有登录时，默认为用户分配匿名用户的权限



ExceptionTranslationFilter

- 处理filterSecurityInterceptor中抛出的异常，然后将请求重定向到对应页面，或返回应用的错误代码



SessionManagementFilter

- 在用户登录成功之后，销毁用户的当前session，并重新生成一个session



filterSecurityInterceptor

- 用户的权限控制都包含在这个过滤器中
- 第一个功能，如果用户尚未登录，抛出尚未认证的异常
- 第二个功能，如果用户已登录，但是没有访问当前资源的权限，会抛出拒绝访问的异常
- 第三个功能，如果用户已登录，也具有访问当前资源的权限，那么放行



FilterChainProxy

- 按照顺序调用一组filter,使他们既能完成验证授权的本职工作，又能相应spring Ioc的功能来很方便地得到其他依赖的资源



## 权限缓存

Spring Security的权限缓存和数据库管理有关。都是在用户认证上做文章。因此都与UserDetailsService有关。与数据库管理不同的是，Spring Security提供了一个实现了可以缓存UserDetailsService的实现类，叫做 CachingUserDetailsService。代码实现如下：

```java
public class CachingUserDetailsService implements UserDetailsService {
    private UserCache userCache = new NullUserCache();
    private final UserDetailsService delegate;

    CachingUserDetailsService(UserDetailsService delegate) {
        this.delegate = delegate;
    }

    public UserCache getUserCache() {
        return userCache;
    }

    public void setUserCache(UserCache userCache) {
        this.userCache = userCache;
    }

    public UserDetails loadUserByUsername(String username) {
        UserDetails user = userCache.getUserFromCache(username);

        if (user == null) {
            user = delegate.loadUserByUsername(username);
        }

        Assert.notNull(user, () -> "UserDetailsService " + delegate
                + " returned null for username " + username + ". "
                + "This is an interface contract violation");

        userCache.putUserInCache(user);

        return user;
    }
}
```

该类的构造函数接受了一个用于真正加载UserDetails的UserDetailsService实现类。当需要加载UserDetails时候，会首先从缓存中获取，如果缓存中没有对应的UserDetails存在则使用持久的UserDetailsService实现类进行加载。然后将加载的结果存放在缓存中。UserDetails与缓存的交互通过UserCache。
 CachingUserDetailsService默认拥有UserCache的空引用实现，叫做NullUserCache。我们可以看到当缓存中不存在对应的UserDetails时，将使用引用的UserDetailsService类型的delegate进行加载。加载后再把它存放在userCache中，并进行返回。
 除了NullUserCache之外，Spring Security还为我们提供了一个基于EHCache的UserCache实现类（EhCacheBasedUserCache）。

```java
public class EhCacheBasedUserCache implements UserCache, InitializingBean {
    // ~ Static fields/initializers
    // =====================================================================================

    private static final Log logger = LogFactory.getLog(EhCacheBasedUserCache.class);

    // ~ Instance fields
    // ================================================================================================

    private Ehcache cache;

    // ~ Methods
    // ========================================================================================================

    public void afterPropertiesSet() throws Exception {
        Assert.notNull(cache, "cache mandatory");
    }

    public Ehcache getCache() {
        return cache;
    }

    public UserDetails getUserFromCache(String username) {
        Element element = cache.get(username);

        if (logger.isDebugEnabled()) {
            logger.debug("Cache hit: " + (element != null) + "; username: " + username);
        }

        if (element == null) {
            return null;
        }
        else {
            return (UserDetails) element.getValue();
        }
    }

    public void putUserInCache(UserDetails user) {
        Element element = new Element(user.getUsername(), user);

        if (logger.isDebugEnabled()) {
            logger.debug("Cache put: " + element.getKey());
        }

        cache.put(element);
    }

    public void removeUserFromCache(UserDetails user) {
        if (logger.isDebugEnabled()) {
            logger.debug("Cache remove: " + user.getUsername());
        }

        this.removeUserFromCache(user.getUsername());
    }

    public void removeUserFromCache(String username) {
        cache.remove(username);
    }

    public void setCache(Ehcache cache) {
        this.cache = cache;
    }
}
```

当我们需要对UserDetails进行缓存时，我们只需要定义一个EHCache的实例，然后把他注入到EhCacheBasedUserCache就可以了。
 刚才介绍的两个类都是Spring Security 已经实现了的。在实际项目中，为了能更好的使用及控制缓存，我们会尝试引用更多的cache。我们不止会缓存UserCache。还会缓存用户相关的权限。我们使用的也不止是内存基本的cache,我们还会使用redis,memcached等来做权限缓存。实际项目在做权限控制时，我们一般会选择自己对相关的数据做缓存。这就相当于我们要实现一个类似于CachingUserDetailsService的类。在我们自己实现的类中，我们可以做更多的扩展。比如，对缓存时间的动态管理，对缓存内容的动态管理，以及对缓存占用空间的动态管理等等。后面我们自己实现一套权限管理的理念，我们会为大家介绍当前最为流行的缓存组件Redis,并在java中封装Redis进行权限缓存。缓存在实际项目中特别重要。原理比较简单，学习起来也相对容易，这里大家一定要学习好，并在项目中使用好。



## 决策管理

决策管理器，其接口为AccessDecisionManager，抽象类为AbstractAccessDecisionManager。而我们要自定义决策管理器的话一般是继承抽象类而不去直接实现接口。

在Spring中引入了投票器（AccessDecisionVoter）的概念，有无权限访问的最终觉得权是由投票器来决定的，最常见的投票器为RoleVoter，在RoleVoter中定义了权限的前缀

Spring提供了3个决策管理器

AffirmativeBased 一票通过，只要有一个投票器通过就允许访问

ConsensusBased 有一半以上投票器通过才允许访问资源

UnanimousBased 所有投票器都通过才允许访问


### Security适配器

自定义类继承WebSecurityConfigurerAdapter(全面接管则在类上加上@EnableWebSecurity注解)

重写以下两个方法实现自己的安全配置

```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception {}
protected void configure(HttpSecurity http) throws Exception {}
```



### 用户认证

通过在configure(AuthenticationManagerBuilder auth)完成用户认证

使用AuthenticationManagerBuilder的inMemoryAuthentication()方法可以添加用户，并给用户指定权限

```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication().withUser("bin").password("123456").roles("ADMIN");
}
```



### 用户授权

通过configure(HttpSecurity http)完成用户授权

HttpSecurity的authorizeRequest()方法有多个子节点，每个matcher按照他们的声明顺序执行，指定用户可以访问的多个URL模式

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/").permitAll()
        .anyRequest().authenticated()
        .and()
        .logout().permitAll()
        .and()
        .formLogin();
    http.csrf().disable();
}
```



### 核心类

**Authentication**

用来表示用户认证信息，用户的相关信息都被封装在一个Authentication对象中

**SecurityContextHolder**

用来保存SecurityContext的，SecurityContext中含有当前所访问系统的用户的详细信息。默认情况下，SecurityContextHolder将使用ThreadLocal来保存SecurityContext，这也就意味着在处于同一线程的方法中，可以从ThreadLocal获取到当前的SecurityContext

**UserDetails**

其中定义了一些可以获取用户名，密码，权限等与认证相关的信息的方法。

可以实现自己的UserDetails，定义我们需要的用户的其他信息

UserDetails是通过UserDetailsService的loadUserByUsername()方法来进行加载的

**UserDetailsService**

Authentication.getPrincipal()的返回类型是Object，但很多情况下返回的其实是一个UserDetails的实例。

登录认证的时候通过UserDetailsService的loadUserByUsername()方法获取对应的UserDetails进行认证，认证通过后会将该UserDetails赋给认证通过的Authentication的principal，然后再把该Authentication存入SecurityContext

**GrantedAuthority**

Authentication.getAuthorities()可以返回当前对象拥有的权限，即当前用户拥有的权限

其返回值是一个GrantedAuthority类型的数组，每一个GrantedAuthority对象代表赋予给当前用户的一种权限

其通常是通过UserDetailsService进行加载的，然后赋予UserDetails的

**DaoAuthenticationProvider**

进行用户认证的处理

在进行认证的时候需要一个UserDetailsService来获取用户的信息

**PasswordEncoder**

对密码的加密



### 验证机制

由一堆Filter实现的，Filter会在Sping MVC前拦截请求，Filter再交由其他组件完成细分的功能



### 登陆验证流程源码解析

Filter->构造Token->AuthenticationManager->转给Provider处理->认证处理成功后续操作或者不通过抛异常

**UsernamePasswordAuthenticationFilter **extends AbstractAuthenticationProcessingFilter

```java
//默认的用户名和密码
public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";
//请求路径和请求方式
public UsernamePasswordAuthenticationFilter() {
    super(new AntPathRequestMatcher("/login", "POST"));
}

public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    if (this.postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    } else {
        //从request中获取参数
        String username = this.obtainUsername(request);
        String password = this.obtainPassword(request);
        if (username == null) {
            username = "";
        }

        if (password == null) {
            password = "";
        }

        username = username.trim();
        //构造一个未认证的Token
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
        //顺便把请求和Token存起来
        this.setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
```

**AbstractAuthenticationProcessingFilter**

该抽象类定义了验证成功与验证失败的处理方法

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest)req;
    HttpServletResponse response = (HttpServletResponse)res;
    if (!this.requiresAuthentication(request, response)) {
        chain.doFilter(request, response);
    } else {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Request is to process authentication");
        }

        Authentication authResult;
        try {
            //调用父类实现的方法进行登录验证
            authResult = this.attemptAuthentication(request, response);
            if (authResult == null) {
                return;
            }

            this.sessionStrategy.onAuthentication(authResult, request, response);
        } catch (InternalAuthenticationServiceException var8) {
            this.logger.error("An internal error occurred while trying to authenticate the user.", var8);
            //验证失败
            this.unsuccessfulAuthentication(request, response, var8);
            return;
        } catch (AuthenticationException var9) {
            //验证失败
            this.unsuccessfulAuthentication(request, response, var9);
            return;
        }

        if (this.continueChainBeforeSuccessfulAuthentication) {
            chain.doFilter(request, response);
        }
		//验证成功
        this.successfulAuthentication(request, response, chain, authResult);
    }
}

protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Authentication success. Updating SecurityContextHolder to contain: " + authResult);
    }
	//验证成功后将Authentication存入SecurityContext
    SecurityContextHolder.getContext().setAuthentication(authResult);
    this.rememberMeServices.loginSuccess(request, response, authResult);
    if (this.eventPublisher != null) {
        this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
    }
	//验证成功后由successHandler决定如何页面的跳转
    this.successHandler.onAuthenticationSuccess(request, response, authResult);
}
```

验证成功之后的Handler和验证失败之后的handler
当我们需要自定义验证成功或失败的处理方法时，要去实现AuthenticationSuccessHandler或AuthenticationfailureHandler接口

下面是一个自定义SuccessHandler例子

类中定义的RedirectStrategy和重写的handle()方法在AbstractAuthenticationTargetUrlRequestHandler中

```java
@Component
public class AppAuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler{	
	// Spring Security 通过RedirectStrategy对象负责所有重定向事务
	private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();

	//重写handle方法，方法中通过RedirectStrategy对象重定向到指定的url
	@Override
	protected void handle(HttpServletRequest request, HttpServletResponse response,
			Authentication authentication)
			throws IOException {
		// 通过determineTargetUrl方法返回需要跳转的url 
		String targetUrl = determineTargetUrl(authentication);
		// 重定向请求到指定的url
		redirectStrategy.sendRedirect(request, response, targetUrl);
	}

	/*
	 * 从Authentication对象中提取角色提取当前登录用户的角色，并根据其角色返回适当的URL。
	 */
	protected String determineTargetUrl(Authentication authentication) {
		String url = "";
		// 获取当前登录用户的角色权限集合
		Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
		List<String> roles = new ArrayList<String>();
		// 将角色名称添加到List集合
		for (GrantedAuthority a : authorities) {
			roles.add(a.getAuthority());
		}

		// 判断不同角色跳转到不同的url
		if (isAdmin(roles)) {
			url = "/admin";
		} else if (isUser(roles)) {
			url = "/home";
		} else {
			url = "/accessDenied";
		}
		System.out.println("url = " + url);
		return url;
	}
	private boolean isUser(List<String> roles) {
		if (roles.contains("ROLE_USER")) {
			return true;
		}
		return false;
	}
	private boolean isAdmin(List<String> roles) {
		if (roles.contains("ROLE_ADMIN")) {
			return true;
		}
		return false;
	}
	public void setRedirectStrategy(RedirectStrategy redirectStrategy) {
		this.redirectStrategy = redirectStrategy;
	}
	protected RedirectStrategy getRedirectStrategy() {
		return redirectStrategy;
	}
}
```

在上面的UsernamePasswordAuthenticationFilter中的方法后面出现了一个UsernamePasswordAuthenticationToken

下面对这个类中涉及到的方法进行说明

```java
//这个构造方法用来初始化一个没有认证的Token实例
public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
    super((Collection)null);
    this.principal = principal;
    this.credentials = credentials;
    this.setAuthenticated(false);
}

//这个构造方法用来初始化一个已经认证的Token实例，为啥要多此一举，不能直接Set状态么，不着急，往后看
public UsernamePasswordAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
    super(authorities);
    this.principal = principal;
    this.credentials = credentials;
    super.setAuthenticated(true);
}

public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
    if (isAuthenticated) {
        //如果是Set认证状态，就无情的给一个异常，意思是：
        //不要在这里设置已认证，不要在这里设置已认证，不要在这里设置已认证
        //应该从构造方法里创建，别忘了要带上用户信息和权限列表哦
        //原来如此，是避免犯错吧
        throw new IllegalArgumentException("Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
    } else {
        super.setAuthenticated(false);
    }
}
```

下面是登录验证内部细节

**ProviderManager**

ProviderManager继承于AuthenticationManager是登录验证的核心类

ProviderManager保管了多个AuthenticationProvider，用于不同类型的登录验证

```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    Class<? extends Authentication> toTest = authentication.getClass();
    AuthenticationException lastException = null;
    AuthenticationException parentException = null;
    Authentication result = null;
    Authentication parentResult = null;
    boolean debug = logger.isDebugEnabled();
    Iterator var8 = this.getProviders().iterator();

    //遍历所有的AuthenticationProvider
    while(var8.hasNext()) {
        AuthenticationProvider provider = (AuthenticationProvider)var8.next();
        if (provider.supports(toTest)) {
            if (debug) {
                logger.debug("Authentication attempt using " + provider.getClass().getName());
            }
            try {
                //进入验证
                result = provider.authenticate(authentication);
                if (result != null) {
                    this.copyDetails(authentication, result);
                    break;
                }
            } catch (AccountStatusException var13) {
                this.prepareException(var13, authentication);
                throw var13;
            } 
            ...
        }
    }
}
```

AuthenticationManager会注册多种AuthenticationProvider

例如UsernamePassword对应的DaoAuthenticationProvider，既然有多种选择，那怎么确定使用哪个Provider呢？

```java
//里面有一个supports方法，返回时一个boolean值，参数是一个Class，没错，这里就是根据Token的类来确定用什么Provider来处理
//我们可以看回前面UsernamePasswordAuthenticationFilter类中出现的下面代码
//UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
//return this.getAuthenticationManager().authenticate(authRequest);
public interface AuthenticationProvider {
    Authentication authenticate(Authentication var1) throws AuthenticationException;
    boolean supports(Class<?> var1);
}
```

**DaoAuthenticationProvider**  extends AbstractUserDetailsAuthenticationProvider

```java
protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
    this.prepareTimingAttackProtection();
    try {
        UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
        if (loadedUser == null) {
            throw new InternalAuthenticationServiceException("UserDetailsService returned null, which is an interface contract violation");
        } else {
            return loadedUser;
        }
    } catch (UsernameNotFoundException var4) {
        this.mitigateAgainstTimingAttack(authentication);
        throw var4;
    } catch (InternalAuthenticationServiceException var5) {
        throw var5;
    } catch (Exception var6) {
        throw new InternalAuthenticationServiceException(var6.getMessage(), var6);
    }
}

//检验密码
protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
    if (authentication.getCredentials() == null) {
        this.logger.debug("Authentication failed: no credentials provided");
        throw new BadCredentialsException(this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
    } else {
        String presentedPassword = authentication.getCredentials().toString();
        if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
            this.logger.debug("Authentication failed: password does not match stored value");
            throw new BadCredentialsException(this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
        }
    }
}
```

**AbstractUserDetailsAuthenticationProvider**

```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    
    //需要UsernamePasswordAuthenticationToken
    Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication, () -> {
        return this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports", "Only UsernamePasswordAuthenticationToken is supported");
    });
    //取出Token里保存的值
    String username = authentication.getPrincipal() == null ? "NONE_PROVIDED" : authentication.getName();
    boolean cacheWasUsed = true;
    //从缓存取
    UserDetails user = this.userCache.getUserFromCache(username);
    if (user == null) {
        cacheWasUsed = false;

        try {
            //没缓存，使用父类的retrieveUser方法获取
            user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication);
        } catch (UsernameNotFoundException var6) {
            this.logger.debug("User '" + username + "' not found");
            if (this.hideUserNotFoundExceptions) {
                throw new BadCredentialsException(this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
            }

            throw var6;
        }

        Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");
    }

    try {
        this.preAuthenticationChecks.check(user);
        this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
    } catch (AuthenticationException var7) {
        if (!cacheWasUsed) {
            throw var7;
        }

        cacheWasUsed = false;
        user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication);
        this.preAuthenticationChecks.check(user);
        this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
    }

    this.postAuthenticationChecks.check(user);
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }

    Object principalToReturn = user;
    if (this.forcePrincipalAsString) {
        principalToReturn = user.getUsername();
    }
    return this.createSuccessAuthentication(principalToReturn, authentication, user);
}

protected Authentication createSuccessAuthentication(Object principal, Authentication authentication, UserDetails user) {
    UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(principal, authentication.getCredentials(), this.authoritiesMapper.mapAuthorities(user.getAuthorities()));
    result.setDetails(authentication.getDetails());
    return result;
}
```



