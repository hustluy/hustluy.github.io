---
layout:     post
title:      "python json.dumps函数输出gbk编码的中文字符串"
date:       2018-01-06 20:00:00
author:     "luy"
catalog: true
tags:
    - python
    - json
    - dumps
---

### 解决办法
json.dumps(obj, ensure\_ascii=False)

### 详细解释
先来看看json.dumps中参数ensure\_ascii的说明, 官方文档给出解释是：  
如果ensure\_ascii=True, 在输出结果中的非ascii字符都会转成\uXXXX形式  
如果ensure\_ascii=False, 非ascii字符可以保持原样  

具体细节可以看下源码：  
```python
if self.ensure_ascii:
    return encode_basestring_ascii(o)
else:
    return encode_basestring(o)
```

encode\_basestring()和encode\_basestring\_ascii()实现:
```python
"""
    PS: 以下代码是从json包源码复制过来的
"""
ESCAPE = re.compile(r'[\x00-\x1f\\"\b\f\n\r\t]')
ESCAPE_ASCII = re.compile(r'([\\"]|[^\ -~])')
HAS_UTF8 = re.compile(r'[\x80-\xff]')
ESCAPE_DCT = {
    '\\': '\\\\',
    '"': '\\"',
    '\b': '\\b',
    '\f': '\\f',
    '\n': '\\n',
    '\r': '\\r',
    '\t': '\\t',
}
for i in range(0x20):
    ESCAPE_DCT.setdefault(chr(i), '\\u{0:04x}'.format(i))
    #ESCAPE_DCT.setdefault(chr(i), '\\u%04x' % (i,))

def encode_basestring(s):
    """Return a JSON representation of a Python string

    """
    def replace(match):
        return ESCAPE_DCT[match.group(0)]
    return '"' + ESCAPE.sub(replace, s) + '"'

def encode_basestring_ascii(s):
    """Return an ASCII-only JSON representation of a Python string

    """
    if isinstance(s, str) and HAS_UTF8.search(s) is not None:
        s = s.decode('utf-8')
    def replace(match):
        s = match.group(0)
        try:
            return ESCAPE_DCT[s]
        except KeyError:
            n = ord(s)
            if n < 0x10000:
                return '\\u{0:04x}'.format(n)
                #return '\\u%04x' % (n,)
            else:
                # surrogate pair
                n -= 0x10000
                s1 = 0xd800 | ((n >> 10) & 0x3ff)
                s2 = 0xdc00 | (n & 0x3ff)
                return '\\u{0:04x}\\u{1:04x}'.format(s1, s2)
                #return '\\u%04x\\u%04x' % (s1, s2)
    return '"' + str(ESCAPE_ASCII.sub(replace, s)) + '"'
```
