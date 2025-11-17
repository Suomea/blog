
请求路径不存在
默认状态码 404，可以处理的异常信息：`NoResourceFoundException`。

请求方法不支持
默认状态码 405，可以处理的异常信息：`HttpRequestMethodNotSupportedException`。

请求体过大

请求体不存在
使用了 @ReuqestBody 注解，但是请求体为空。默认状态码 400，可以处理的异常信息：`HttpMessageNotReadableException`。

认证失败 
401

鉴权失败 
403

`@Valid` 是 JSR-303 标准注解，适用于基本的校验需求，不支持分组校验。
`@Validated` 是 Spring 扩展注解，支持分组校验，适用于需要针对不同场景应用不同校验规则的情况。

常用的注解：
```
The annotated element must not be null. Accepts any type.
@NotNull

The annotated element must not be null and must contain at least one non-whitespace character. Accepts CharSequence.
@NotBlank

The annotated element must not be null nor empty.
Supported types are:
	CharSequence (length of character sequence is evaluated)
	Collection (collection size is evaluated)
	Map (map size is evaluated)
	Array (array length is evaluated)
@NotEmpty

The annotated element must be a number whose value must be higher or equal to the specified minimum.
Supported types are:
	BigDecimal
	BigInteger
	byte, short, int, long, and their respective wrappers
	Note that double and float are not supported due to rounding errors (some providers might provide some approximative support).
null elements are considered valid.
@Min

The annotated element must be a number whose value must be lower or equal to the specified maximum.
Supported types are:
	BigDecimal
	BigInteger
	byte, short, int, long, and their respective wrappers
	Note that double and float are not supported due to rounding errors (some providers might provide some approximative support).
null elements are considered valid.
@Max

The string has to be a well-formed email address. Exact semantics of what makes up a valid email address are left to Jakarta Bean Validation providers. Accepts CharSequence.
null elements are considered valid.
@Email

The annotated CharSequence must match the specified regular expression. The regular expression follows the Java regular expression conventions see java.util.regex.Pattern.
Accepts CharSequence. 
null elements are considered valid.
@Pattern

The annotated element size must be between the specified boundaries (included).
Supported types are:
	CharSequence (length of character sequence is evaluated)
	Collection (collection size is evaluated)
	Map (map size is evaluated)
	Array (array length is evaluated)
null elements are considered valid.
@Size
```

引入依赖
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```

添加注解
```java
@PostMapping("/dto")  
public void dto(@RequestBody @Valid TestDTO dto) {  
}
```

校验不通过，默认状态码 400，可以处理的异常信息：`MethodArgumentNotValidException`。