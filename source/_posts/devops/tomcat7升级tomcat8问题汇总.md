## tomcat7升级tomcat8问题汇总

#### 1、tomcat的日志文件权限与启动用户的权限不一致
用户work的文件权限(umask=0002)为

u=rwx,g=rwx,o=rx

但是tomcat的日志文件的权限却是
u=rw,g=r,o=---

为什么会不一样呢？

这是因为tomcat在启动(catalina.sh)时会重新设置UMASK，（其默认值为0027，根操作系统的默认值0022不一样），去那个脚本里修改一下即可，如下：

```shell
# set UMASK unless it has been ovverriden
if [ -z "$UMASK" ]; then 
	UMASK = `umask`
#	UMASK = "0027"
fi
umask $UMASK
```


改成系统当前用户的umask即可。

#### 2、Tomcat 从7升级到8的时候出现了java.lang.IllegalArgumentException: An inval id domain [.xxx.com] was specified for this cookie 异常。

在 tomcat context.xml中配置 <CookieProcessor className="org. apache .tomcat.util.http.LegacyCookieProcessor" /> 即可，或者你也可以把域名前面的 . 去掉。

分析Tomcat 8更换了默认的 CookieProcessor 实现为 Rfc6265CookieProcessor ，之前的实现为 LegacyCookieProcessor 。前者是基于 RFC6265 ，而后者基于 RFC6265 、 RFC2109 、 RFC2616 。
RFC6265 中关于domain有这么一段描述：


```
Domain=domain  Optional. The Domain attribute specifies the domain for which the  cookie is valid. An explicitly specified domain must always start  with a dot.
```

而在 RFC6265 
中：

```
If the attribute-name case-insensitively matches the string “Domain”,  the user agent MUST process the cookie-av as follows.
If the attribute-value is empty, the behavior is undefined. However,  the user agent SHOULD ignore the cookie-av entirely.
If the first character of the attribute-value string is %x2E (“.”):
Let cookie-domain be the attribute-value without the leading %x2E  (“.”) character.
Otherwise:
Let cookie-domain be the entire attribute-value.
Convert the cookie-domain to lower case.
```
一个说要 . 

一个说不要，那再来看看具体 代码 实现（generateHeader方法）。

LegacyCookieProcessor 
（省略了部分代码）

```java

@Override
public String generateHeader(Cookie cookie) {
        ......
        String domain = cookie.getDomain();
        ......
        // Add domain information, if present
        if (domain != null) {
                buf.append("; Domain=");
                maybeQuote(buf, domain, version);
        }
        ......
}
```

再看看maybeQuote方法实现：

```
private void maybeQuote(StringBuffer buf, String value, int version) {
        if (value == null || value.length() == 0) {
            buf.append("/"/"");
        } else if (alreadyQuoted(value)) {
            buf.append('"');
            escapeDoubleQuotes(buf, value,1,value.length()-1);
            buf.append('"');
        } else if (needsQuotes(value, version)) {
            buf.append('"');
            escapeDoubleQuotes(buf, value,0,value.length());
            buf.append('"');
        } else {
            buf.append(value);
        }
}
```
可以看到并没有做任何domain校验的操作；

Rfc6265CookieProcessor 
：

```java
@Override
public String generateHeader(Cookie cookie) {
        ......
        String domain = cookie.getDomain();
        if (domain != null && domain.length() > 0) {
            validateDomain(domain);
            header.append("; Domain=");
            header.append(domain);
        }
        ......
}
private void validateDomain(String domain) {
        int i = 0;
        int prev = -1;
        int cur = -1;
        char[] chars = domain.toCharArray();
        while (i < chars.length) {
            prev = cur;
            cur = chars[i];
            if (!domainValid.get(cur)) {
                throw new IllegalArgumentException(sm.getString(
                        "rfc6265CookieProcessor.invalidDomain", domain));
            }
            // labels must start with a letter or number
            // RFC6265实现
            if ((prev == '.' || prev == -1) && (cur == '.' || cur == '-')) {
                throw new IllegalArgumentException(sm.getString(
                        "rfc6265CookieProcessor.invalidDomain", domain));
            }
            // labels must end with a letter or number
            // RFC6265实现
            if (prev == '-' && cur == '.') {
                throw new IllegalArgumentException(sm.getString(
                        "rfc6265CookieProcessor.invalidDomain", domain));
            }
            i++;
        }
        // domain must end with a label
        if (cur == '.' || cur == '-') {
            throw new IllegalArgumentException(sm.getString(
                    "rfc6265CookieProcessor.invalidDomain", domain));
        }
}

```
validateDomain方法中实现了 RFC6265 


