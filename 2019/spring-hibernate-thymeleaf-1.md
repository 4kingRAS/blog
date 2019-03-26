# Springboot + Spring security + hibernate开发 (1)
*在海航的IT部门短暂实习一个月，给的任务是一个数据库小项目，后台前端都要做。虽然比较低端，但也是一个后端程序员的基础，花一个月时间是值得的。用Springboot技术栈完成。这里记录一下流程和遇到的大坑小坑*

### 1 - 开发环境部署
&emsp;&emsp;首先java项目最好还是用IDEA全家桶，方便，自动化程度高。Springboot的方便之处就在于简化了各种配置，让你专心写CRUD，集成了tomcat，最后生成jar包，而不用手动打包。

&emsp;&emsp;开发前，准备好需要的文档，还是得看官方文档，但是官方文档直接看很头疼，比较好的方式是看其他人的博客教程，然后对照着官方文档看。官方推荐做法也不一定是最好的，比如security推荐角色权限写死在后台，就很不灵活。不一定跟他一样。
[Springboot Reference](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#getting-started-system-requirements)

&emsp;&emsp;用IDEA全家桶很简单，直接spring initializer生成springboot的项目就行，在项目依赖勾选需要的组件， jpa，security等等。
或者直接在maven里配置：
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity4</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```
&emsp;&emsp;然后配置数据库，很简单，参考官方教程，我这里是用hibernate操作数据库，国内一般喜欢mysql+mybatis。
[Accessing data with mysql](https://spring.io/guides/gs/accessing-data-mysql/)
照着此教程一路做下去即可，没什么问题。

```java
#第一次创建实体表用create，以后用update none都行
spring.jpa.hibernate.ddl-auto=update

#前端热启动
spring.thymeleaf.cache=false

#控制台显示对应的sql语句，方便调试
spring.jpa.show-sql=true
server.port=8080

#解决中文查询
spring.datasource.url=jdbc:mysql://localhost:3306/db_supply?useUnicode=true&characterEncoding=utf-8
```

### 2 - MVC结构搭建

&emsp;&emsp;首先搭建数据库，然后建好实体类。实体类对应的DAO,DAO最好再对应一个Service,扩展性更好，但也更啰嗦。看自己取舍。

最后的结构大概就是这样：
```
controller:
    - UserController.java
entity:
    - User.java
service:
    - UserDAO.java
    _ UserService.java  #这两个都是接口
    implement:
    - UserServiceImpl.java
config:
    - xxx
```
DAO里jpa自动完成了crud接口，但有些查询需要自己写明:
```java
public interface UserDAO extends JpaRepository<User, Integer> {
    public User findByUsername(String username);
}
```

实体类需要注意的是一些注解问题，注解的检查都是操作数据库时的检查，最好还是在前端进行检查，用户体验好：

```java
@Entity 
//@Tabel(name = 'User')如果类名与表名不一致才需要注解 
public class User implements Serializable {
 
    // GenerationType： identity每个表独立自增id
    // auto有可能是公用一个id
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) 
    @Column(name = "user_id")
    private Integer id;

    @NotEmpty(message = "invalid username") 
    // notempty只对字符串有用 NotNull可检查int等
    @Column(name = "user_name", length = 20, nullable = false) // 字段长度
    private String username;
    //.....
}
```

关于Controller注解：

&emsp;&emsp;@RestController是@ResponseBody和@Controller的组合注解
@Controller 需要返回相应的html模版
如果需要返回json，比如下面的带参数返回就需要注解@ResponseBody

```java
@Controller
@RequestMapping(value = "/user")
public class UserController {

    @Autowired
    UserService userService;

    Utils utils;

    /**
     *  获取用户列表
     */
    @RequestMapping(method = RequestMethod.GET)
    public String getUserList(ModelMap map) {
        map.addAttribute("userList", userService.findAllUser());
        return "userList";
    }

    /**
     * 显示创建用户表单
     *
     * @param map 添加属性
     * @return 成功页面 在html中
     */
    @RequestMapping(value = "/reg", method = RequestMethod.GET)
    public String createUserForm(ModelMap map) {
        User user = new User();
        map.addAttribute("user", user);
        map.addAttribute("action", "reg");
        return "regForm";
    }

    /**
     *  创建用户
     *    处理 "/users" 的 POST 请求，用来获取用户列表
     *    通过 @ModelAttribute 绑定参数，也通过 @RequestParam 从页面中传递参数
     */
    @RequestMapping(value = "/reg", method = RequestMethod.POST)
    public String postUserForm(ModelMap map,
                           @ModelAttribute @Valid User user,
                           BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            map.addAttribute("action", "reg");
            return "regForm";
        }

        try {
            Date d = new Date();
            Timestamp t = new Timestamp(d.getTime());

            user.setGroup(1);
            user.setIsOnline("1");
            user.setLastLogin(t);
            user.setIsActived("1");

            userService.addSingleUser(user);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return "redirect:/user/";  //跳转而不是返回html模版，跳转实际上是调用相关url的controller
    }
}
```

view端，现在流行前后端分离开发，由于我们做的是小demo暂不考虑这些，学习前端也很费功夫。

简单的html如下： ${key}即thymeleaf的模版特性，controller层要有一个model给key的赋值

```html
<!DOCTYPE html>
<!-- test spring boot -->
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>注册页面</title>

<style>
    p.head {
        color: #555555;
        text-align: center;
        font-size: 25px;
        font-family: "Georgia";
    }

    p.text {
        color: #666666;
        text-align: center;
        font-size: 20px;
        font-family: "Georgia";
    }
</style>

</head>

<body>

<p class="head">spring boot<br><span th:text="${key}"></span></p>
<form action="/demo/add" method="post">
    <p class="text">Name: <input type="text" name="name" value="Mickey"></p>
    <p class="text">Email: <input type="text" name="email" value="default@abc.com"></p>
    <p class="text">Password: <input type="password" name="password" value="Mouse"></p>
    <div style = "text-align:center;"><input type="submit" value=" submit "></div>
</form>

</body>

</html>
```

简陋的登录页面完成，下期介绍spring security的坑。

