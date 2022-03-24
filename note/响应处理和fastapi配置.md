## 响应状态码和快捷属性

```python
import uvicorn
from fastapi import FastAPI
from starlette import status

app = FastAPI()


@app.post("/status_code", status_code=200)
async def status_code():
    return {"status_code": 200}


@app.post("/status_attribute", status_code=status.HTTP_200_OK)
async def status_attribute():
    print(type(status.HTTP_200_OK))
    return {"status_code": status.HTTP_200_OK}


if __name__ == '__main__':
    uvicorn.run('main:app', reload=True, debug=True, workers=1)
```

## 表单数据处理

