BEC-PLAT Tutorial & Demo
==========================
@version: v1 | @author: shiziye | @date:2014.5.26


1. setup basic environment
2. build a simple openstack webapp (web page/rest api)
3. share some component & practice
4. begin to write code

Part I. Environment Setup
======================
1. JDK
	* install [jdk-7u45-windows-x64](download/jdk-7u45-windows-x64.exe)
	* set environment variable
	
			JAVA_HOME = C:\Program Files\Java\jdk1.7.0_45
			PATH = %PATH%;%JAVA_HOME%\bin

2. Maven
	* download [apache-maven-3.0.5-bin.zip](download/apache-maven-3.0.5-bin.zip)
	* unzip to "c:\maven-3.0.5"
	* set environment variable

			M2_HOME = C:\maven-3.0.5
			M2 = %M2_HOME%\bin
			PATH = %PATH%;%M2%
	* (Optional) baidu PMO maven repository global setting
		* add [setting.xml](download/setting.xml) to c:\%homepath%\.m2\
		* about setting.xml [http://wiki.babel.baidu.com/twiki/bin/view/Com/Pmo/Scm/Maven_user](http://wiki.babel.baidu.com/twiki/bin/view/Com/Pmo/Scm/Maven_user)
3. IntelliJ (IDE)
	* install [ideaIC-13.1.2.exe](download/ideaIC-13.1.2.exe)
	* set jdk/maven path
	* community VS Ultimate
		* html、css、javascript assistent
		* velocity assistent
		* FE will use WebStorm/Chrome
4. HotSwap (IntelliJ)
	* with mouse
		1. debug mode
		2. Run(Menu)->Reload Change Class
	* with keyboard
		1. Settings->Debugger->HotSwap: set "Always"
		2. Just Press "Ctrl + F9", will invoke save->compile->reload

Part II. First Application
======================

1. prepare source code
	* download & unzip [bec-plat-demo-step2.zip](demo/bec-plat-demo-step2.zip)
2. Play in IntelliJ
	* Open it with IntelliJ
		1. File->Open.. -> choose the folder contains "pom.xml"
		2. IntelliJ will auto generate `.idea/` & `bec-plat-demo.iml`
	* Run
		1. bec-plat-demo/scr/main/java/com.baidu.bec.plat.dashboard/`Application.java`
		2. right click -> `Run "Application.main()"`
		3. open chrome [http://localhost:8080](http://localhost:8080)
	* Debug
		1. add break
		2. debug mode
	* UnitTest
		1. right click `bec-plat-demo[plat-dashboard]`
		2. click `Run 'All Tests'`
3. Play in Console
	* mvn (quick test)
	
			mvn spring-boot:run
	
	* single jar (deploy)
	
			mvn package
			java -Dserver.port=80 -jar target/XXX.jar

4. Application Files 
	* pom.xml
		* groupId, artifactId, version 
		* spring dependency
		* velocity dependency
		* Baidu Repository
	* src/main
		* /java
			* /com.baidu.bec.plat.dashboard
				* Application.java (main)
				* WelcomeController (Controller & Model)
		* /resources
			* /template
				* welcome.vm (View, velocity)
			* application.properties (config)
				* spring.velocity.cache: false (hotswap) 
			* logback.xml (log config)
			* /static (jpg, css, javascript ……）
	* src/test
		* /java/com.baidu.bec.plat.Application.jar (Web UnitTest)
	* target (maven & IntelliJ compile output)


Part III. OpenStack DEMO (Basic WebPage) 
==================================
1. Parepare openstack virtual machine
	* Follow [local_deploy.html](../local_deploy.html)
	* Finish
		* install `VirtualBox`
		* download `CentOS 6.3 X64 minimal with OpenStack Basic Install v0.2 2014.5.18`
		* create Virtual Machine
		* config Virtual Network
		* start OpenStack Virtual Machine 
	* login as root， and stop iptables to allow access
		* service iptables stop
2. OpenStack WebPage 

1) Use [openstack4j (fluent API)](http://www.openstack4j.com/)

2) Add dependency

		<dependency>
			<groupId>org.pacesys</groupId>
			<artifactId>openstack4j</artifactId>
			<version>1.0.0</version>
		</dependency>

3) Add src/main/java/com/baidu/bec/play/dashboard/`OpenStackController.java`

	@Controller
	public class OpenStackController {
	
	    @Value("${keystone.endpoint:http://192.168.56.10:5000/v2.0}")
	    String endpoint = "http://192.168.56.10:5000/v2.0";
	
	    @RequestMapping("/os/list")
	    public String list(Map<String, Object> model) {
	
	        OSClient os = OSFactory.builder()
	                .endpoint(endpoint)
	                .credentials("admin","admin")
	                .tenantName("admin")
	                .authenticate();
	
	        model.put("users", os.identity().users().list());
	        model.put("tenants", os.identity().tenants().list());
	        model.put("flavors", os.compute().flavors().list());
	        model.put("images", os.images().list());
	        return "os/list";
	    }
	}

	
4) `Alt+Enter` auto import pakcage namespace
	
5) Add src/main/resources/templates/os/list.vm

	<!DOCTYPE html>
	
	<html lang="en">
	<body>
	    <div>
	        <span>Users</span>
	        <ul>
	    	    #foreach ($user in $users)
	    		<li>${user.name}</li>
	    		#end
	    	</ul>
		</div>
		<div>
		    <span>Tenants</span>
	    	<ul>
	    	    #foreach ($tenant in $tenants)
	    		<li>${tenant.name}</li>
	    		#end
	    	</ul>
	    </div>
	    <div>
	    	<span>Flavors</span>
	        <ul>
	            #foreach ($flavor in $flavors)
	        	<li>${flavor.name} Vcpus:${flavor.vcpus} Ram:${flavor.ram}</li>
	        	#end
	        </ul>
	    </div>
	    <div>
	    	<span>Images</span>
	        <ul>
	       	    #foreach ($image in $images)
	       		<li>${image.name}</li>
	      		#end
	       	</ul>
	    </div>
	
	</body>
	</html>

6) Add `keystone.endpoint` to `application.properties`

	keystone.endpoint: http://192.168.56.10:5000/v2.0

7) Run `Application.main()` & Browse [http://localhost:8080/os/list](http://localhost:8080/os/list)

8) Snapshot of Above Step:  [bec-plat-demo-step3.zip](demo/bec-plat-demo-step3.zip)

Part IV. OpenStack DEMO (Param/Exception & REST API) 
========================

1) Show Param/Exception, Add `listWithUserPassword` method 

* @PathVariable
* @RequestParam

-


    @RequestMapping("/os/list/{user}")
    public String listWithUserPassword(
            @PathVariable String user,
            @RequestParam(defaultValue = "wrong_password") String password,
            Map<String, Object> model) {

        OSClient os;
        try {
            os = OSFactory.builder()
                    .endpoint(endpoint)
                    .credentials(user, password)
                    .tenantName("admin")
                    .authenticate();
        }catch (Exception ex){
            throw new UserPasswordException();
        }

        model.put("users", os.identity().users().list());
        model.put("tenants", os.identity().tenants().list());
        model.put("flavors", os.compute().flavors().list());
        model.put("images", os.images().list());
        return "os/list";
    }

2) Add `UserPasswordExcetpion`

* @ResponseStatus

-

    @ResponseStatus(value = HttpStatus.UNAUTHORIZED, reason="username or password wrong")
    public class UserPasswordException extends RuntimeException { }

3) Browse [http://localhost:8080/os/list/admin?password=admin](http://localhost:8080/os/list/admin?password=admin)



4) Show REST API, Add `api_list` method


* @ResponseBody
* @ModuleAttribute

-

    @RequestMapping(value = "/api/list", method = RequestMethod.GET)
    @ResponseBody
    public Information api_list(@ModelAttribute UserPassword userPassword) {
        OSClient os;
        try {
            os = OSFactory.builder()
                    .endpoint(endpoint)
                    .credentials(userPassword.getUser(), userPassword.getPassword())
                    .tenantName("admin")
                    .authenticate();
        }catch (Exception ex){
            throw new UserPasswordException();
        }

        List<? extends User> users = os.identity().users().list();
        List<? extends Tenant> tenants = os.identity().tenants().list();
        List<? extends Flavor> flavors = os.compute().flavors().list();
        List<? extends Image> images = os.images().list();
        return new Information(users, tenants, flavors, images);
    }

5) Add `UserPassword` model

* only need to wirte 2 private field (user, password)
* IntelliJ can auto generate getter/setter 

-
    
	public static class UserPassword {
        private String user;
        private String password;

		// IntelliJ generate these for us
        public String getUser() {
            return user;
        }
        public String getPassword() {
            return password;
        }
        public void setUser(String user) {
            this.user = user;
        }
        public void setPassword(String password) {
            this.password = password;
        }
    }

6) Add `Information` model

* only need to write 4 private field (users, tenants, flavors, images)
* IntelliJ can auto generate constructor & getter

-

    public static class Information{

        private List<? extends User> users;
        private List<? extends Tenant> tenants;
        private List<? extends Flavor> flavors;
        private List<? extends Image> images;

		// IntelliJ generate these for us
        public Information(List<? extends User> users, List<? extends Tenant> tenants, List<? extends Flavor> flavors, List<? extends Image> images) {
            this.users = users;
            this.tenants = tenants;
            this.flavors = flavors;
            this.images = images;
        }
        public List<? extends User> getUsers() {
            return users;
        }
        public List<? extends Tenant> getTenants() {
            return tenants;
        }
        public List<? extends Flavor> getFlavors() {
            return flavors;
        }
        public List<? extends Image> getImages() {
            return images;
        }
    }

7) Browse [http://localhost:8080/api/list?user=admin&password=admin](http://localhost:8080/api/list?user=admin&password=admin)

8) add json-pretty-print to `application.properties` & reload

	http.mappers.json-pretty-print: true

9) Snapshot of Above Step:  [bec-plat-demo-step4.zip](demo/bec-plat-demo-step4.zip)


From here, hard to to show demo, review only
--------------------------------------------------
Part V. Auzhentication with UUAP(cas) && Log
==========================================

1) Add cas security dependencies in PMO

	<!-- cas security begin -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-cas</artifactId>
        <version>3.2.3.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-taglibs</artifactId>
        <version>3.2.3.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
    </dependency>
     <!-- cas security end-->

2) Add WebSecurityConfig.java to config cas

	@Configuration
	@EnableWebSecurity
	public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	
	    @Value("${cas.server.loginUrl:http://itebeta.baidu.com:8100/login}")
	    private String casServerLoginUrl = "http://itebeta.baidu.com:8100/login";
	
	    @Value("${cas.server.logoutUrl:http://itebeta.baidu.com:8100/logout}")
	    private String casServerLogoutUrl = "http://itebeta.baidu.com:8100/login";
	
	    @Value("${cas.server.validateUrl:http://itebeta.baidu.com:8100}")
	    private String casServerValidateUrl = "http://itebeta.baidu.com:8100";
	
	    @Value("${cas.client.serviceUrl:http://localhost:8080/j_spring_cas_security_check}")
	    private String casClientServiceUrl = "http://localhost:8080/j_spring_cas_security_check";
	
	    private String casAuthenticationKey = "any_cas_key";
	
	    @Override
	    protected void configure(HttpSecurity http) throws Exception {
	        http
	            .addFilter(casAuthenticationFilter())
	            .authorizeRequests()
	                .antMatchers("/api/**").permitAll()
	                .anyRequest().authenticated()
	                .and()
	            .logout()
	                .logoutSuccessUrl(casServerLogoutUrl)
	                .invalidateHttpSession(true)
	                .and()
	            .exceptionHandling()
	                .accessDeniedPage("/acceesDenied")
	                .authenticationEntryPoint(casAuthenticationEntryPoint());
	    }
	
	    @Override
	    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
	        auth
	                .authenticationProvider(casAuthenticationProvider());
	    }
	
	    @Bean
	    public ServiceProperties serviceProperties() {
	        ServiceProperties serviceProperties = new ServiceProperties();
	        serviceProperties.setService(casClientServiceUrl);
	        serviceProperties.setSendRenew(false);
	        return serviceProperties;
	    }
	
	    @Bean
	    public CasAuthenticationProvider casAuthenticationProvider() {
	        CasAuthenticationProvider casAuthenticationProvider = new CasAuthenticationProvider();
	        casAuthenticationProvider.setAuthenticationUserDetailsService(authenticationUserDetailsService());
	        casAuthenticationProvider.setServiceProperties(serviceProperties());
	        casAuthenticationProvider.setTicketValidator(cas20ServiceTicketValidator());
	        casAuthenticationProvider.setKey(casAuthenticationKey);
	        return casAuthenticationProvider;
	    }
	    @Bean
	    public Cas20ServiceTicketValidator cas20ServiceTicketValidator() {
	        return new Cas20ServiceTicketValidator(casServerValidateUrl);
	    }
	

	    @Bean
	    public CasAuthenticationEntryPoint casAuthenticationEntryPoint() {
	        CasAuthenticationEntryPoint casAuthenticationEntryPoint = new CasAuthenticationEntryPoint();
	        casAuthenticationEntryPoint.setLoginUrl(casServerLoginUrl);
	        casAuthenticationEntryPoint.setServiceProperties(serviceProperties());
	        return casAuthenticationEntryPoint;
	    }
	
	    @Bean
	    public CasAuthenticationFilter casAuthenticationFilter() throws Exception {
	        CasAuthenticationFilter casAuthenticationFilter = new CasAuthenticationFilter();
	        casAuthenticationFilter.setAuthenticationManager(authenticationManager());
	        return casAuthenticationFilter;
	    }
	
	
	    @Bean
	    public AuthenticationUserDetailsService authenticationUserDetailsService() {
	        return new CasAuthenticationUserDetailsService();
	    }

3) Add CasAuthenticationUserDetailsService in WebsecurityConfig class

    @Autowired
    VelocityViewResolver velocityViewResolver;
    class CasAuthenticationUserDetailsService implements AuthenticationUserDetailsService {
        @Override
        public UserDetails loadUserDetails(Authentication token) throws UsernameNotFoundException {
            List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
            authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
            UserDetails userDetails = new User(token.getName(), "", authorities);

            return userDetails;
        }
    }

4) Add spring-security-taglibs velocity authz integration to `VelocityConfiguration.java` 

	@Bean
    Authz authz() {return new AuthzImpl();}

	VelocityViewResolver velocityViewResolver() {
		......

        // set velocity authentication variable
        Properties properties = new Properties();
        properties.put("authz", authz());
        resolver.setAttributes(properties);
	}


5) Use authentication in velocity, add code to "welcome.vm"

    User: ${authz.principal}
    <a href="logout">logout</a>

6) Use pricipal in controller, add code to `WelcomeController.java`

	public String welcome(Principal principal, Map<String, Object> model)

7) Log

* slf4j.logger
* logback.xml config

-

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;

	@Controller
	public class WelcomeController {
	
	    Logger log = LoggerFactory.getLogger(getClass());
	
	    @Value("${application.message}")
		private String message = "Hello World";
	
		@RequestMapping("/")
		public String welcome(Principal principal, Map<String, Object> model) {
	        if (log.isInfoEnabled())
	            log.info("[" + principal.getName() + "]" + " access");
	
			model.put("time", new Date());
			model.put("message", this.message);
			return "welcome";
		}
	}


8) Snapshot of Above Step  [bec-plat-demo-step5.zip](demo/bec-plat-demo-step5.zip)

Part VI. More
=================================

* velocity.layout
* Global Exception Handler
* profile (debug/production)
* java doc
* Keystone Token


What To Do Next
================
* Read Spring Boot Guide (source code)
* Read Spring Reference
* Read Velocity Guide
* Follow Java Style (google)
* Use OpenStack
* Agile & Refactor
* We Build prototype system in 1 week (most openstack controll feature)
* We setup Technology Stack in 3 month
	* authentication
	* log
	* error handling
	* unittest
	* layout ([common/functions]/[template,controller,service,extra])
	* doc generation
	* site-map & swagger
	* metrix
	* Tools Chain: 
		* Hotswap: spring-loaded / Jrebel
