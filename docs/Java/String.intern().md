

代码里的解释
```java
    /**
     * Returns a canonical representation for the string object.
     * <p>
     * A pool of strings, initially empty, is maintained privately by the
     * class {@code String}.
     * <p>
     * When the intern method is invoked, if the pool already contains a
     * string equal to this {@code String} object as determined by
     * the {@link #equals(Object)} method, then the string from the pool is
     * returned. Otherwise, this {@code String} object is added to the
     * pool and a reference to this {@code String} object is returned.
     * <p>
     * It follows that for any two strings {@code s} and {@code t},
     * {@code s.intern() == t.intern()} is {@code true}
     * if and only if {@code s.equals(t)} is {@code true}.
     * <p>
     * All literal strings and string-valued constant expressions are
     * interned. String literals are defined in section 3.10.5 of the
     * <cite>The Java&trade; Language Specification</cite>.
     *
     * @return  a string that has the same contents as this string, but is
     *          guaranteed to be from a pool of unique strings.
     */
    public native String intern();
```

intern 方法的作用是将字符串添加到字符串常量池中，如果字符串常量池中已经包含该字符串，那么直接返回字符串常量池中的字符串引用；如果字符串常量池中没有该字符串，则将字符串复制到字符串常量池中，并返回字符串常量池中的引用。

需要注意的是 new String("hello") 永远是在堆中创建一个新的对象，和字符串常量池没有关系（虽然 JDK 7 及之后字符串常量池也在堆中）。

字符串字面量如 `"hello"、"world"` 和字符串常量表达式（可以在编译时确定值的字符串表达式）如 `"hello" + "world"` 都会被内部化，存储在字符串常量池中。

示例1
```java
String s1 = "hello"; // "hello" 被添加到字符串常量池
String s2 = "hello"; // 从字符串常量池中获取引用
String s3 = new String("hello"); // 在堆中创建一个新对象

System.out.println(s1 == s2); // true，引用相同
System.out.println(s1 == s3); // false，引用不同
```

示例2
```java
String s1 = new String("hello"); // 在堆中创建一个新对象
String s2 = s1.intern(); // 将 "hello" 添加到字符串常量池，并返回引用
String s3 = "hello"; // 从字符串常量池中获取引用

System.out.println(s1 == s2); // false，s1 是堆中的对象，s2 是常量池中的引用
System.out.println(s2 == s3); // true，引用相同
```

