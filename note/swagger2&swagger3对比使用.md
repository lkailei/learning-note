## Swagger2&Swagger对比

### 2.0版本

**引入pom**

```java
<!-- swagger工具包 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>${swagger.version}</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>${swagger.version}</version>
</dependency>
```

**配置**

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Value("${swagger.show}")
    private boolean swaggerShow;

    @Bean
    public Docket swaggerSpringMvcPlugin() {
        return new Docket(DocumentationType.SWAGGER_2)
                .enable(swaggerShow)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.springboot.springbootswagger"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger2 演示接口RESTful APIs")
                .version("1.0")
                .build();
    }
}
```

Swagger 注解详细说明：

| **API注解**        | **作用范围**         | **使用位置**                       | 参数                                                         |
| ------------------ | -------------------- | ---------------------------------- | ------------------------------------------------------------ |
| @ApiModel          | 用于参数为实体类的中 | 用在返回对象类上                   | value="用途作用"，description="描述"                         |
| @ApiModelProperty  | 对象属性             | 用在出入参数对象的字段上           | value="名称"，name="名称"，notes="备注"                      |
| @Api               | 协议集描述           | 用于 controller 类上               | tags="说明该类的作用，可以在UI界面上看到的注解"     value="该参数没什么意义，在UI界面上也看到，所以不需要配置" |
| @ApiOperation      | 协议描述             | 用在接口的方法上  tag用于接口分组  | value="说明方法的用途、作用"     notes="方法的备注说明"      |
| @ApiResponses      | Response集           | 用在接口的方法上                   |                                                              |
| @ApiResponse       | Response             | 用在 @ApiResponses 里边            | code：数字，例如400         message：信息，例如"请求参数没填好"         response：抛出异常的类 |
| @ApiImplicitParams | 非对象参数集         | 用在接口的的方法上                 |                                                              |
| @ApiImplicitParam  | 非对象参数描述       | 用在 @ApiImplicitParams 的方法里边 | name：参数名         value：参数的汉字说明、解释         required：参数是否必须传         paramType：参数放在哪个地方             · header --> 请求参数的获取：@RequestHeader             · query --> 请求参数的获取：@RequestParam             · path（用于restful接口）--> 请求参数的获取：@PathVariable             · body（不常用）             · form（不常用）             dataType：参数类型，默认String，其它值dataType="Integer"                defaultValue：参数的默认值 |

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ApiModel(value = "用户演示类", description = "请求参数类")
public class User {
    @ApiModelProperty(example = "1", notes = "用户ID")
    private Long id;
    @ApiModelProperty(example = "geekdigging", notes = "用户名")
    private String nickName;
    @ApiModelProperty(example = "1570689455000", notes = "创建时间")
    private Date createDate;
    @ApiModelProperty(example = "18", notes = "用户年龄")
    private Integer age;
}	


	@GetMapping("/getUserById")
	@ApiOperation(value = "根据id获取用户", notes = "id必传", httpMethod = "GET")
	@ApiImplicitParam(name = "id", value = "用户id",example = "1", required = true, dataType = "long", paramType = "query")
	public UserEntity getOne(Long id) {
		return userDao.getOne(id);
	}

	@PostMapping("/getUserByNameAndSex")
	@ApiOperation(value = "根据name和sex获取用户", notes = "", httpMethod = "POST")
	@ApiImplicitParams({
			@ApiImplicitParam(name = "userName", value = "用户名", example = "关羽", required = true, dataType = "string", paramType = "query"),
			@ApiImplicitParam(name = "userSex", value = "用户性别", example = "男", required = true, dataType = "string", paramType = "query") })
	public UserEntity getUserByNameAndSex(String userName, String userSex) {
		return userDao.getUserByNameAndSex(userName, userSex);
	}
```

文档接口地址：http://localhost:8080/v2/api-docs
文档页面地址：http://localhost:8080/swagger-ui.html

### 3.0版本

**引入pom**

```java
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-boot-starter -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
@Configuration
// @EnableSwagger2 无需使用该注解
public class SwaggerConfiguration {
 
    @Bean
    public Docket createRestApi() {
        // 创建 Docket 对象
        return new Docket(DocumentationType.SWAGGER_2) // 文档类型，使用 Swagger2
                .apiInfo(this.apiInfo()) // 设置 API 信息
                // 扫描 Controller 包路径，获得 API 接口
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.iocoder.springboot.lab24.controller"))
                .paths(PathSelectors.any())
                // 构建出 Docket 对象
                .build();
    }
 
    /**
     * 创建 API 信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试接口文档示例")
                .description("我是一段描述")
                .version("1.0.0") // 版本号
                .build();
    }
 
```

文档接口地址：http://localhost:8080/v3/api-docs
文档页面地址：http://localhost:8080/swagger-ui/index.html

| **作用域** | **2.0版本注解**   | **3.0版本注解** |
| ---------- | ----------------- | --------------- |
| 类对象     | @ ApiModel        | @Schema         |
| 属性       | @ApiModelProperty | @Schema         |
| 属性       | @ApiModelProperty | @ArraySchema    |
| 类         | @Api              | @Tag            |
| 方法       | @ApiOperation     | @Operation      |
| 方法       | @ApiResponses     | @ApiResponses   |
| 方法       | @ApiResponse      | @ApiResponse    |

```java
@Data
@Schema(name = "首页头部信息", description = "首页头部信息")
public class IndexHeadInfoVo implements Serializable {

    @Schema(example = "搜索框内展示的内容(默认展示最新开班的课程名称)", type = "string")
    private String searchClassName;

    @Schema(example = "账户数据信息", type = "object")
    private AccountInfo accountInfo;

    @ArraySchema(schema = @Schema(name = "IndexSubjectVo", description = "首页科目数据信息"))
    private List<IndexSubjectVo> subjectList;
}
@Tag(name = "test-controller",description = "相关控制器")
public class TestController {

    @Autowired
    private ITestService iTestService;

    @PostMapping(value = "/test")
    @Operation(description = "测试",summary = "测试",tags = "test-controller")
    public List<CampusLocationVo> test() {
        return iTestService.test();
    }
    
    
}
```

新版本和老版本的区别主要体现在以下 4 个方面：

1. **依赖**项的添加不同：新版本只需要添加一项，而老版本需要添加两项；
2. 启动 Swagger 的注解不同：新版本使用的是 **@EnableOpenApi**，而老版本是 @EnableSwagger2；
3. Docket（文档摘要信息）的文件类型配置不同：新版本配置的是 **OAS_3**，而老版本是 SWAGGER_2

```java
PS： 1：Swagger 可以看作是一个遵循了 OpenAPI 规范的一项技术，而 springfox Swagger 则是这项技术的具体实现 
2：OAS 是 OpenAPI Specification 的简称，翻译成中文就是 OpenAPI 说明书。
```

1. Swagger UI 访问地址不同：新版本访问地址是“**http://localhost:8080/swagger-ui/**”，而老版本访问地址是“http://localhost:8080/swagger-ui.html”