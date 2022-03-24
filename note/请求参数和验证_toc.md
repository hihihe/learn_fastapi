<a name="index">**Index**</a>  
&emsp;<a href="#0">url通用格式</a>  
&emsp;<a href="#1">127.0.0.1与0.0.0.0</a>  
&emsp;<a href="#2">fastapi交互式文档</a>  
&emsp;<a href="#3">路径参数和数据的解析、验证</a>  
&emsp;&emsp;<a href="#4">通过url传递参数parameter</a>  
&emsp;&emsp;<a href="#5">通过url传递枚举参数</a>  
&emsp;&emsp;<a href="#6">通过url传递文件路径参数</a>  
&emsp;&emsp;<a href="#7">路径参数数字验证-通过Path类限制路径参数</a>  
&emsp;<a href="#8">查询参数</a>  
&emsp;&emsp;<a href="#9">查询参数字符串验证-通过Query类限制查询参数</a>  
&emsp;<a href="#10">请求体和字段-通过Field类限制请求体字段</a>  
&emsp;<a href="#11">路径参数+查询参数+请求体</a>  
&emsp;<a href="#12">数据格式嵌套的请求体</a>  
&emsp;<a href="#13">如何设置Cookie和Header</a>  
## <a name="0">url通用格式</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

<协议>://<用户名>:<密码>@<主机域名或者ip地址>:<端口号>/<路径>;<参数>?<查询>#<片段>

```
http://joe:password@www.baidu.com:80/main/index.html;type=a;color=b?name=bob&id=123#main
```

[参考链接](https://blog.csdn.net/luluoluoa/article/details/115616220)

## <a name="1">127.0.0.1与0.0.0.0</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

127.0.0.1 是一个环回地址。并不表示“本机”。0.0.0.0才是真正表示“本网络中的本机”。

在实际应用中，一般我们在服务端绑定端口的时候可以选择绑定到0.0.0.0，这样我的服务访问方就可以通过我的多个ip地址访问我的服务。

比如我有一台服务器，一个外放地址A,一个内网地址B，如果我绑定的端口指定了0.0.0.0，那么通过内网地址或外网地址都可以访问我的应用。

## <a name="2">fastapi交互式文档</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

![image-20220323200217491](https://chushi123.oss-cn-beijing.aliyuncs.com/img/202203232002545.png)

![image-20220323192317946](https://chushi123.oss-cn-beijing.aliyuncs.com/img/202203231923047.png)

## <a name="3">路径参数和数据的解析、验证</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

路径参数：在url中传递参数

### <a name="4">通过url传递参数parameter</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()  # 这里不一定是app，名字随意，运行时需保持一致


@app.get("/path/{parameter}")  # 函数的顺序就是路由的顺序
def path_prams(parameter: str):
    return {"message": parameter}


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

### <a name="5">通过url传递枚举参数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
from enum import Enum

import uvicorn
from fastapi import FastAPI

app = FastAPI()  # 这里不一定是app，名字随意，运行时需保持一致


class CityName(str, Enum):
    Beijing = "Beijing China"
    Shanghai = "Shanghai China"


@app.get("/enum/{city}")  # 枚举类型的参数
async def latest(city: CityName):
    if city == CityName.Shanghai:
        return {"city_name": city, "confirmed": 1492, "death": 7}
    if city == CityName.Beijing:
        return {"city_name": city, "confirmed": 971, "death": 9}


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

### <a name="6">通过url传递文件路径参数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()  # 这里不一定是app，名字随意，运行时需保持一致


@app.get("/files/{file_path:path}")  # 通过 参数:path 传递文件路径
def filepath(file_path: str):
    return f"The file path is {file_path}"


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

### <a name="7">路径参数数字验证-通过Path类限制路径参数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
import uvicorn
from fastapi import FastAPI, Path

app = FastAPI()  # 这里不一定是app，名字随意，运行时需保持一致


@app.get("/path/{num}")
def path_params_validate(
        num: int = Path(..., title="Your Number", description="数字", ge=1, le=10),
):
    return num


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

## <a name="8">查询参数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
from typing import Optional

import uvicorn
from fastapi import FastAPI

app = FastAPI()  # 这里不一定是app，名字随意，运行时需保持一致


@app.get("/query")
def page_limit(page: int = 1, limit: Optional[int] = None):  # 给了默认值就是选填的参数，没给默认值就是必填参数
    """可以通过请求http://127.0.0.1:8000/query?page=1验证"""
    if limit:
        return {"page": page, "limit": limit}
    return {"page": page}


@app.get("/query/bool/conversion")  # bool类型转换：yes on 1 True true会转换成true, 其它为false
def type_conversion(param: bool = False):
    """可以通过请求http://127.0.0.1:8000/query/bool/conversion?param=false验证"""
    return param


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

### <a name="9">查询参数字符串验证-通过Query类限制查询参数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
from typing import List

import uvicorn
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/query/validations")  # 长度+正则表达式验证，比如长度8-16位，以a开头。其它校验方法看Query类的源码
def query_params_validate(
        value: str = Query(..., min_length=8, max_length=16, regex="^a"),  # ... 表示必填参数，...换成None就变成选填的参数
        values: List[str] = Query(["v1", "v2"], alias="alias_name")  # alias给参数起别名，本来参数名values，现在是alias_name
):  # 多个查询参数的列表。参数别名
    return value, values


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

## <a name="10">请求体和字段-通过Field类限制请求体字段</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
import uvicorn
from fastapi import FastAPI
from pydantic import Field, BaseModel

app = FastAPI()


class CityInfo(BaseModel):
    name: str = Field(..., example="Beijing")  # example是注解的作用，值不会被验证
    country: str
    country_code: str = None  # 给一个默认值
    country_population: int = Field(default=800, title="人口数量", description="国家的人口数量", ge=800)

    class Config:
        schema_extra = {
            "example": {
                "name": "Shanghai",
                "country": "China",
                "country_code": "CN",
                "country_population": 1400000000,
            }
        }


@app.post("/request_body/city")
def city_info(city: CityInfo):
    print(city.name, city.country)  # 当在IDE中输入city.的时候，属性会自动弹出
    return city.dict()


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

## <a name="11">路径参数+查询参数+请求体</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
import uvicorn
from fastapi import FastAPI, Body, Query
from pydantic import Field, BaseModel

app = FastAPI()


class CityInfo(BaseModel):
    name: str = Field(..., example="Beijing")  # example是注解的作用，值不会被验证
    country: str
    country_code: str = None  # 给一个默认值
    country_population: int = Field(default=800, title="人口数量", description="国家的人口数量", ge=800)

    class Config:
        schema_extra = {
            "example": {
                "name": "Shanghai",
                "country": "China",
                "country_code": "CN",
                "country_population": 1400000000,
            }
        }


@app.put("/request_body/city/{name}")
def mix_city_info(
        name: str,  # 路径参数
        city01: CityInfo,  # 请求体
        city02: CityInfo,  # 请求体Body可以是多个的
        confirmed: int = Query(ge=0, description="确诊数", default=0),  # 查询参数
        death: int = Query(ge=0, description="死亡数", default=0),
):
    if name == "Shanghai":
        return {"Shanghai": {"confirmed": confirmed, "death": death}}
    return city01.dict(), city02.dict()


@app.put("/request_body/multiple/parameters")
def body_multiple_parameters(
        city: CityInfo = Body(..., embed=True),  # 当只有一个Body参数的时候，embed=True表示请求体参数嵌套。多个Body参数默认就是嵌套的
        confirmed: int = Query(ge=0, description="确诊数", default=0),
        death: int = Query(ge=0, description="死亡数", default=0),
):
    print(f"{city.name} 确诊数：{confirmed} 死亡数：{death}")
    return city.dict()


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

## <a name="12">数据格式嵌套的请求体</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
from datetime import date
from typing import List

import uvicorn
from fastapi import FastAPI
from pydantic import Field, BaseModel

app = FastAPI()


class CityInfo(BaseModel):
    name: str = Field(..., example="Beijing")  # example是注解的作用，值不会被验证
    country: str
    country_code: str = None  # 给一个默认值
    country_population: int = Field(default=800, title="人口数量", description="国家的人口数量", ge=800)

    class Config:
        schema_extra = {
            "example": {
                "name": "Shanghai",
                "country": "China",
                "country_code": "CN",
                "country_population": 1400000000,
            }
        }


class Data(BaseModel):
    city: List[CityInfo] = None  # 这里就是定义数据格式嵌套的请求体
    date: date  # 额外的数据类型，还有uuid datetime bytes frozenset等，参考：https://fastapi.tiangolo.com/tutorial/extra-data-types/
    confirmed: int = Field(ge=0, description="确诊数", default=0)
    deaths: int = Field(ge=0, description="死亡数", default=0)
    recovered: int = Field(ge=0, description="痊愈数", default=0)


@app.put("/request_body/nested")
def nested_models(data: Data):
    return data


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

## <a name="13">如何设置Cookie和Header</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

```python
from typing import List, Optional

import uvicorn
from fastapi import FastAPI, Cookie, Header

app = FastAPI()


@app.get("/cookie")  # 效果只能用Postman测试
def cookie(cookie_id: Optional[str] = Cookie(None)):  # 定义Cookie参数需要使用Cookie类，否则就是查询参数
    return {"cookie_id": cookie_id}


@app.get("/header")
def header(user_agent: Optional[str] = Header(None, convert_underscores=True), x_token: List[str] = Header(None)):
    """
    有些HTTP代理和服务器是不允许在请求头中带有下划线的，所以Header提供convert_underscores属性让设置
    :param user_agent: convert_underscores=True 会把 user_agent 变成 user-agent
    :param x_token: x_token是包含多个值的列表
    :return:
    """
    return {"User-Agent": user_agent, "x_token": x_token}


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```