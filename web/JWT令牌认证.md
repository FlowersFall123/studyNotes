# **JWT令牌认证**

## 1. 基础配置

### 1.1 启用Servlet组件扫描

在Spring Boot主类添加`@ServletComponentScan`注解：

```
@SpringBootApplication
@ServletComponentScan
public class TestEnd01Application {
    public static void main(String[] args) {
        SpringApplication.run(TestEnd01Application.class, args);
    }
}
```

### 1.2 添加Maven依赖

在`pom.xml`中添加：

```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>2.0.53</version>
</dependency>
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.8.2</version>
</dependency>
```

## 2. JWT工具类实现

### 2.1 JWTUtil类

```
public class JWTUtil {
    private static final String SECRET = "your-secret-key"; // 加密密钥
    private static final long EXPIRATION = 86400000; // 1天有效期
    
    // 创建Token
    public static String createToken(User user) {
        Date expireDate = new Date(System.currentTimeMillis() + EXPIRATION * 7);
        return JWT.create()
            .withHeader(Map.of("alg", "HS256", "typ", "JWT"))
            .withClaim("id", user.getId())
            .withClaim("username", user.getUsername())
            .withClaim("password", "secret")
            .withExpiresAt(expireDate)
            .withIssuedAt(new Date())
            .sign(Algorithm.HMAC256(SECRET));
    }
    
    // 验证Token
    public static Map<String, Claim> verifyToken(String token) {
        try {
            return JWT.require(Algorithm.HMAC256(SECRET))
                     .build()
                     .verify(token)
                     .getClaims();
        } catch (Exception e) {
            logger.error("Token验证失败", e);
            return null;
        }
    }
}
```

## 3. JWT过滤器实现

### 3.1 JWTFilter类

```
@Slf4j
@WebFilter(filterName = "JWTFilter", urlPatterns = "/api/*")
public class JWTFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) 
        throws IOException, ServletException {
        
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        String requestURI = request.getRequestURI();
        
        // 放行认证相关路径
        if (requestURI.startsWith("/api/auth")) {
            chain.doFilter(request, response);
            return;
        }
        
        // 处理OPTIONS请求
        if ("OPTIONS".equals(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);
            chain.doFilter(request, response);
            return;
        }
        
        // 验证Token
        String token = request.getHeader("authorization");
        if (token == null) {
            response.getWriter().write(JSON.toJSONString(RestBean.failure(401,"未提供token")));
            return;
        }
        
        Map<String, Claim> userData = JWTUtil.verifyToken(token);
        if (userData == null) {
            response.getWriter().write(JSON.toJSONString(RestBean.failure(401,"token不合法")));
            return;
        }
        
        // 设置用户属性到请求中
        request.setAttribute("id", userData.get("id").asInt());
        request.setAttribute("username", userData.get("username").asString());
        request.setAttribute("password", userData.get("password").asString());
        
        chain.doFilter(req, res);
    }
}
```

## 4. 登录接口改造

### 4.1 登录成功后返回Token

```
@PostMapping("/auth/login")
public RestBean<String> login(@RequestBody User user) {
    // 验证用户逻辑...
    String token = JWTUtil.createToken(user);
    return RestBean.success(token);
}
```

## 5. 前端集成

### 5.1 获取Token的辅助函数

```
function getAuthToken() {
    return localStorage.getItem('authToken') || '';
}
```

### 5.2 请求拦截器配置

```
// 在axios请求拦截器中添加Token
axios.interceptors.request.use(config => {
    config.headers["Authorization"] = getAuthToken();
    return config;
});
```

### 5.3 登录成功后存储Token

```
// 登录成功回调
localStorage.setItem("authToken", response.data);
```

## 6. 效果验证

1. **未登录状态**：
   - 访问受保护API返回401未授权
   - 控制台显示"未提供token"或"token不合法"
2. **登录状态**：
   - 成功获取Token并存储在localStorage
   - 后续请求自动携带Token头
   - 可以正常访问受保护API

## 注意事项

1. 密钥(SECRET)应足够复杂且妥善保管
2. Token有效期应根据业务需求设置
3. 生产环境应使用HTTPS传输Token
4. 考虑实现Token刷新机制
5. 敏感操作应进行二次验证