# SpringBoot整合jcasbin

## mavan仓库引入
```
<dependency>  
    <groupId>org.casbin</groupId>  
    <artifactId>jcasbin</artifactId>  
    <version>1.1.0</version>  
</dependency>  
<dependency>  
    <groupId>org.casbin</groupId>  
    <artifactId>jdbc-adapter</artifactId>  
    <version>1.1.0</version>  
</dependency>  
```

## 配置文件
jcasbin把用户的角色、权限信息（访问路径）放置在配置文件里，然后通过输入流读取配置文件。主要有两个配置文件：model.conf 和 policy.csv。简单的使用GitHub里都讲了，在此就不再赘述了。

其实也可以读取数据库的角色权限配置。所以我们可以把关于数据库的信息提取出来，可以进行动态设置。
```
@Configuration  
@ConfigurationProperties(prefix = "org.jcasbin")  
public class EnforcerConfigProperties {  
   
 private String url;  
   
 private String driverClassName;  
   
 private String username;  
   
 private String password;  
   
 private String modelPath;  
   
 public String getUrl() {  
  return url;  
 }  
   
 public void setUrl(String url) {  
  this.url = url;  
 }  
   
 public String getDriverClassName() {  
  return driverClassName;  
 }  
   
 public void setDriverClassName(String driverClassName) {  
  this.driverClassName = driverClassName;  
 }  
   
 public String getUsername() {  
  return username;  
 }  
   
 public void setUsername(String username) {  
  this.username = username;  
 }  
   
 public String getPassword() {  
  return password;  
 }  
   
 public void setPassword(String password) {  
  this.password = password;  
 }  
   
 public String getModelPath() {  
  return modelPath;  
 }  
   
 public void setModelPath(String modelPath) {  
  this.modelPath = modelPath;  
 }  
   
 @Override  
 public String toString() {  
  return "EnforcerConfigProperties [url=" + url + ", driverClassName=" + driverClassName + ", username="  
    + username + ", password=" + password + ", modelPath=" + modelPath + "]";  
 }  
}
```

这样我们就可以在application.properties里进行相关配置了。model.conf是固定的文件，之间复制过来放在新建的和src同级的文件夹下即可。policy.csv的内容是可以从数据库读取的。

```
org.jcasbin.url=jdbc:mysql://localhost:3306/casbin?useSSL=false  
org.jcasbin.driver-class-name=com.mysql.jdbc.Driver  
org.jcasbin.username=root  
org.jcasbin.password=root  
org.jcasbin.model-path=conf/authz_model.conf  
```

## 读取权限信息进行初始化
我们要对Enforcer这个类初始化，加载配置文件里的信息。所以我们写一个类实现InitializingBean，在容器加载的时候就初始化这个类，方便后续的使用。
```
@Component  
public class EnforcerFactory implements InitializingBean {  
   
 private static Enforcer enforcer;  
   
 @Autowired  
 private EnforcerConfigProperties enforcerConfigProperties;  
 private static EnforcerConfigProperties config;  
   
 @Override  
 public void afterPropertiesSet() throws Exception {  
  config = enforcerConfigProperties;  
  //从数据库读取策略  
  JDBCAdapter jdbcAdapter = new JDBCAdapter(config.getDriverClassName(),config.getUrl(),config.getUsername(),  
             config.getPassword(), true);  
  enforcer = new Enforcer(config.getModelPath(), jdbcAdapter);  
  enforcer.loadPolicy();//Load the policy from DB.  
 }  
   
 /**  
  * 添加权限  
  * @param policy  
  * @return  
  */  
 public static boolean addPolicy(Policy policy){  
  boolean addPolicy = enforcer.addPolicy(policy.getSub(),policy.getObj(),policy.getAct());  
  enforcer.savePolicy();  
    
  return addPolicy;  
 }  
   
 /**  
  * 删除权限  
  * @param policy  
  * @return  
  */  
 public static boolean removePolicy(Policy policy){  
  boolean removePolicy = enforcer.removePolicy(policy.getSub(),policy.getObj(),policy.getAct());  
  enforcer.savePolicy();  
    
  return removePolicy;  
 }  
   
 public static Enforcer getEnforcer(){  
  return enforcer;  
 }  
}  
```

在这个类里，我们注入写好的配置类，然后转为静态的，在afterPropertiesSet方法里实例化Enforcer并加载policy（策略，角色权限/url对应关系）。

同时又写了两个方法，用来添加和删除policy，Policy是自定的一个类，对官方使用的集合/数组进行了封装。

```
public class Policy {  
 /**想要访问资源的用户 或者角色*/  
 private String sub;  
   
 /**将要访问的资源，可以使用  * 作为通配符，例如/user/* */  
 private String obj;  
   
 /**用户对资源执行的操作。HTTP方法，GET、POST、PUT、DELETE等，可以使用 * 作为通配符*/  
 private String act;  
   
 public Policy() {  
  super();  
 }  
   
 /**  
  *   
  * @param sub 想要访问资源的用户 或者角色  
  * @param obj 将要访问的资源，可以使用  * 作为通配符，例如/user/*  
  * @param act 用户对资源执行的操作。HTTP方法，GET、POST、PUT、DELETE等，可以使用 * 作为通配符  
  */  
 public Policy(String sub, String obj, String act) {  
  super();  
  this.sub = sub;  
  this.obj = obj;  
  this.act = act;  
 }  
   
 public String getSub() {  
  return sub;  
 }  
   
 public void setSub(String sub) {  
  this.sub = sub;  
 }  
   
 public String getObj() {  
  return obj;  
 }  
   
 public void setObj(String obj) {  
  this.obj = obj;  
 }  
   
 public String getAct() {  
  return act;  
 }  
   
 public void setAct(String act) {  
  this.act = act;  
 }  
   
 @Override  
 public String toString() {  
  return "Policy [sub=" + sub + ", obj=" + obj + ", act=" + act + "]";  
 }  
   
}  
```

## 权限控制

jcasbin的权限控制非常简单，自定义一个过滤器，if判断就可以搞定，没错，就这么简单。

```
@WebFilter(urlPatterns = "/*" , filterName = "JCasbinAuthzFilter")  
@Order(Ordered.HIGHEST_PRECEDENCE)//执行顺序，最高级别最先执行，int从小到大  
public class JCasbinAuthzFilter implements Filter {  
   
 private static final Logger log = LoggerFactory.getLogger(JCasbinAuthzFilter.class);  
   
 private static Enforcer enforcer;  
   
 @Override  
 public void init(FilterConfig filterConfig) throws ServletException {  
 }  
   
 @Override  
 public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)  
   throws IOException, ServletException {  
  HttpServletRequest request = (HttpServletRequest) servletRequest;  
        HttpServletResponse response = (HttpServletResponse) servletResponse;  
          
        String user = request.getParameter("username");  
        String path = request.getRequestURI();  
        String method = request.getMethod();  
   
        enforcer = EnforcerFactory.getEnforcer();  
        if (path.contains("anon")) {  
         chain.doFilter(request, response);  
  }else if (enforcer.enforce(user, path, method)) {  
   chain.doFilter(request, response);  
  } else {  
   log.info("无权访问");  
   Map<String, Object> result = new HashMap<String, Object>();  
            result.put("code", 1001);  
            result.put("msg", "用户权限不足");  
            result.put("data",null);  
            response.setCharacterEncoding("UTF-8");  
            response.setContentType("application/json");  
            response.getWriter().write(JSONObject.toJSONString(result,SerializerFeature.WriteMapNullValue));  
     }  
    
 }  
   
 @Override  
 public void destroy() {  
    
 }  
}  
```

主要是用enforcer.enforce(user, path, method)这个方法对用户、访问资源、方式进行匹配。这里的逻辑可以根据自己的业务来实现。在这个过滤器之前还可以添加一个判断用户是否登录的过滤器。

## 添加删除权限

对于权限的操作，直接调用上面写好的EnforcerFactory里对应的方法即可。并且，可以达到同步的效果。就是不用重启服务器或者其他任何操作，添加或删除用户权限后，用户对应的访问就会收到影响。

```
@PutMapping("/anon/role/per")  
public ResultBO<Object> addPer(){  
   
 EnforcerFactory.addPolicy(new Policy("alice", "/user/list", "*"));  
   
 return ResultTool.success();  
}  
  
@DeleteMapping("/anon/role/per")  
public ResultBO<Object> deletePer(){  
   
 EnforcerFactory.removePolicy(new Policy("alice", "/user/list", "*"));  
   
 return ResultTool.success();  
}  
```