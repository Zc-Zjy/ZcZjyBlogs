---
title: StringUtils中isEmpty和isBank的区别
date: 2023-12-01 13:54:40
tags: StringUtils中isEmpty和isBank的区别
categories: [后端技术,问题方法合集]
---

<br/>

***

<br/>

# 一、isEmpty系列
### 1、StringUtils.isEmpty
``` java
StringUtils.isEmpty(null) = true
StringUtils.isEmpty("") = true
StringUtils.isEmpty(" ") = false
StringUtils.isEmpty(“bob”) = false
StringUtils.isEmpty(" bob ") = false
/
 * 源码
 * <p>NOTE: This method changed in Lang version 2.0.
 * It no longer trims the CharSequence.
 * That functionality is available in isBlank().</p>
 *
 * @param cs  the CharSequence to check, may be null
 * @return {@code true} if the CharSequence is empty or null
 * @since 3.0 Changed signature from isEmpty(String) to isEmpty(CharSequence)
 */
public static boolean isEmpty(final CharSequence cs) {
    return cs == null || cs.length() == 0;
}
```
**说明：**  
是否为空，可以看到 " " 空格是会绕过这种空判断,因为是一个空格,并不是严格的空值,会导致 isEmpty(" ")=false。  

### 2、StringUtils.isNotEmpty
相当于不为空， = !isEmpty()。  
``` java
public static boolean isNotEmpty(final CharSequence cs) {
    return !isEmpty(cs);
}
```

### 3、StringUtils.isAnyEmpty
是否有一个为空，只有一个为空，就为true。
``` java
StringUtils.isAnyEmpty(null) = true
StringUtils.isAnyEmpty(null, “foo”) = true
StringUtils.isAnyEmpty("", “bar”) = true
StringUtils.isAnyEmpty(“bob”, “”) = true
StringUtils.isAnyEmpty(" bob ", null) = true
StringUtils.isAnyEmpty(" ", “bar”) = false
StringUtils.isAnyEmpty(“foo”, “bar”) = false
/  
 * 源码
 * @param css  the CharSequences to check, may be null or empty
 * @return {@code true} if any of the CharSequences are empty or null
 * @since 3.2
 */
public static boolean isAnyEmpty(final CharSequence... css) {
  if (ArrayUtils.isEmpty(css)) {
    return true;
  }
  for (final CharSequence cs : css){
    if (isEmpty(cs)) {
      return true;
    }
  }
  return false;
}
```

### 4、StringUtils.isNoneEmpty
相当于`!isAnyEmpty(css)`，必须所有的值都不为空才返回true。
``` java
StringUtils.isNoneEmpty(null)             = false
StringUtils.isNoneEmpty(null, "foo")      = false
StringUtils.isNoneEmpty("", "bar")        = false
StringUtils.isNoneEmpty("bob", "")        = false
StringUtils.isNoneEmpty("  bob  ", null)  = false
StringUtils.isNoneEmpty(" ", "bar")       = true
StringUtils.isNoneEmpty("foo", "bar")     = true
```


<br/>

***

<br/>


# 二、isBank系列
### 1、StringUtils.isBlank
是否为真空值（空格或者空值）。
``` java
StringUtils.isBlank(null) = true
StringUtils.isBlank("") = true
StringUtils.isBlank(" ") = true
StringUtils.isBlank(“bob”) = false
StringUtils.isBlank(" bob ") = false
/
 * <p>Checks if a CharSequence is whitespace, empty ("") or null.</p>
 * @param cs  the CharSequence to check, may be null
 * @return {@code true} if the CharSequence is null, empty or whitespace
 * @since 2.0
 * @since 3.0 Changed signature from isBlank(String) to isBlank(CharSequence)
 */
public static boolean isBlank(final CharSequence cs) {
    int strLen;
    if (cs == null || (strLen = cs.length()) == 0) {
        return true;
    }
    for (int i = 0; i < strLen; i++) {
        if (Character.isWhitespace(cs.charAt(i)) == false) {
            return false;
        }
    }
    return true;
}
```

### 2、StringUtils.isNotBlank
是否真的不为空，不是空格或者空值，相当于`!isBlank()`。
``` java
public static boolean isNotBlank(final CharSequence cs) {
        return !isBlank(cs);
}
```

### 3、StringUtils.isAnyBlank
是否包含任何真空值（包含空格或空值）。
``` java
StringUtils.isAnyBlank(null) = true
StringUtils.isAnyBlank(null, “foo”) = true
StringUtils.isAnyBlank(null, null) = true
StringUtils.isAnyBlank("", “bar”) = true
StringUtils.isAnyBlank(“bob”, “”) = true
StringUtils.isAnyBlank(" bob ", null) = true
StringUtils.isAnyBlank(" ", “bar”) = true
StringUtils.isAnyBlank(“foo”, “bar”) = false
 /
 * <p>Checks if any one of the CharSequences are blank ("") or null and not whitespace only..</p>
 * @param css  the CharSequences to check, may be null or empty
 * @return {@code true} if any of the CharSequences are blank or null or whitespace only
 * @since 3.2
 */
public static boolean isAnyBlank(final CharSequence... css) {
  if (ArrayUtils.isEmpty(css)) {
    return true;
  }
  for (final CharSequence cs : css){
    if (isBlank(cs)) {
      return true;
    }
  }
  return false;
}
```

### 4、StringUtils.isNoneBlank
是否全部都不包含空值或空格。
``` java
StringUtils.isNoneBlank(null) = false
StringUtils.isNoneBlank(null, “foo”) = false
StringUtils.isNoneBlank(null, null) = false
StringUtils.isNoneBlank("", “bar”) = false
StringUtils.isNoneBlank(“bob”, “”) = false
StringUtils.isNoneBlank(" bob ", null) = false
StringUtils.isNoneBlank(" ", “bar”) = false
StringUtils.isNoneBlank(“foo”, “bar”) = true
/
* <p>Checks if none of the CharSequences are blank ("") or null and whitespace only..</p>
* @param css  the CharSequences to check, may be null or empty
* @return {@code true} if none of the CharSequences are blank or null or whitespace only
* @since 3.2
*/
public static boolean isNoneBlank(final CharSequence... css) {
    return !isAnyBlank(css);
}
```


<br/>

***

<br/>


# 三、StringUtils的其他方法
可以参考官方的文档，里面有详细的描述,有些方法还是很好用的：  
[点击连接跳转](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html)
