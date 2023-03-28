# 模仿chat_pdf
## 原有代码
```python

from xml import etree

import requests
import json
from requests_toolbelt import MultipartEncoder
from flask import Flask, jsonify, request
import json
from flask_cors import CORS

proxy = '127.0.0.1:10808'
proxies = {
    'http': 'socks5://' + proxy,
    'https': 'socks5://' + proxy
}
headers = {
    'Content-Type': 'application/json; charset=UTF-8',
    "user-agent": "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Mobile Safari/537.36"
}
chatpdf_idtoken = "eyJhbGciOiJSUzI1NiIsImtpZCI6Ijk3OWVkMTU1OTdhYjM1Zjc4MjljZTc0NDMwN2I3OTNiN2ViZWIyZjAiLCJ0eXAiOiJKV1QifQ.eyJuYW1lIjoicGVuZyB6ZW5nIiwicGljdHVyZSI6Imh0dHBzOi8vbGgzLmdvb2dsZXVzZXJjb250ZW50LmNvbS9hL0FHTm15eFlUVzhIVmpNSmtVSXJGRXgwSHJucWgwQVZmNFRZTzh5YlhqZ3NnPXM5Ni1jIiwiaXNzIjoiaHR0cHM6Ly9zZWN1cmV0b2tlbi5nb29nbGUuY29tL3RhbGstdG8tYW55dGhpbmciLCJhdWQiOiJ0YWxrLXRvLWFueXRoaW5nIiwiYXV0aF90aW1lIjoxNjc5NzAyMDM5LCJ1c2VyX2lkIjoidHVORkphcnZhME83YWRTOUswSVlZZHNOc0RnMiIsInN1YiI6InR1TkZKYXJ2YTBPN2FkUzlLMElZWWRzTnNEZzIiLCJpYXQiOjE2Nzk3MjMyODUsImV4cCI6MTY3OTcyNjg4NSwiZW1haWwiOiJ6ZW5ncGVuZzIwMjMwMzIzQGdtYWlsLmNvbSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJmaXJlYmFzZSI6eyJpZGVudGl0aWVzIjp7Imdvb2dsZS5jb20iOlsiMTE0Mjk0MDk4OTQxNDM1MDQwNzQzIl0sImVtYWlsIjpbInplbmdwZW5nMjAyMzAzMjNAZ21haWwuY29tIl19LCJzaWduX2luX3Byb3ZpZGVyIjoiZ29vZ2xlLmNvbSJ9fQ.bcyQAVYnHWsMY5stUVjTUfZ3YGtplbdpkhtOQhH-oM4NiK8SFFBLBQVtIvfKBBCfak_8g7mOtdysWTGv7KOqkniDzMsceMPC-_eqohZC5MMamFx2TK61iLtHTPfLjefYEqj4kfRUMXftSgcOOVdEN3KqKrJWIbJ3vgEJ-_1lWLYn1b5p6i-XMLOhRwhytaIH3e6uNyOTSOOmdnRH1pmJJFTgekeKnIoEa2qVzBKEZPEof1N8svKGmTaje5m-XWmWff2gEEKVZAh9aYmEB7h-0toOaaf5xrDTjej_taNIJvUIDdhB7itc_5UE3R-ef1W3KQ2XtMLRv9ktq5nAOGlaHA"

app = Flask(__name__)
CORS(app)


def upload(file):
    url = "https://upload-pr4yueoqha-ue.a.run.app/to-to-anything/us-east1/upload"

    headers = {
        'Content-Type': 'application/json',
        "chatpdf-idtoken": chatpdf_idtoken
    }
    file_name = 'demo.pdf'
    multipart_encoder = MultipartEncoder(
        fields={
            'file': (file_name, open(file, 'rb'), 'application/octet-stream')}
    )
    headers['Content-Type'] = multipart_encoder.content_type

    response = requests.post(url, data=multipart_encoder, headers=headers,
                             proxies=proxies)
    print(response.text)
    sourceId = response.json().get("sourceInfo").get("sourceId")

    headers = {
        'Content-Type': 'application/json',
        "user-agent": "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Mobile Safari/537.36",
        "chatpdf-idtoken": chatpdf_idtoken
    }
    url = "https://analyze-pr4yueoqha-ue.a.run.app/to-to-anything/us-east1/analyze"
    payload = {"sourceId": sourceId}
    response = requests.post(url, data=json.dumps(payload), headers=headers,
                             proxies=proxies)
    print(response.text)
    return sourceId


def talk(my_data):
    headers = {
        'Content-Type': 'application/json; charset=UTF-8',
        "user-agent": "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Mobile Safari/537.36",
        "chatpdf-idtoken": chatpdf_idtoken
    }
    url = "https://chat-pr4yueoqha-ue.a.run.app/to-to-anything/us-east1/chat"

    response = requests.post(url, data=json.dumps(my_data), headers=headers,
                             proxies=proxies)
    response.encoding = response.apparent_encoding
    print(response.text)
    split_data = response.text.split("\n")

    humanMsgId = json.loads(split_data[0]).get("humanMsgId")
    aiMsgId = json.loads(split_data[0]).get("aiMsgId")
    message = response.text[len(split_data[0]):]
    return {"humanMsgId": humanMsgId, "aiMsgId": aiMsgId, "message": message}


@app.route('/hello', methods=['GET'])
def hello():
    return "hello"


@app.route('/upload', methods=['POST'])
def upload_file():
    f = open('workfile.pdf', "wb")
    f.write(request.files.get("file").read())
    f.close()
    file = 'workfile.pdf'
    sourceId = upload(file)
    data = {"sourceId": sourceId}
    return jsonify(data)


@app.route('/task', methods=['POST'])
def task():
    data = talk(json.loads(request.data))
    return jsonify(data)


if __name__ == "__main__":
    app.run(port=8000, host="0.0.0.0", debug=True)
```

上一版本的代码思路是通过 requests 来直接请求 chat_pdf 官方网站，将用户的请求转发给 chat_pdf,
然后将chat_pdf 的响应转发给用户。并没有自己实现 chat_pdf 的功能，只是将 chat_pdf 的功能封装成了一个接口。

---

## 对于 chat_pdf 网站的分析

### 上传 PDF 文件

通过抓包分析，发现上传 PDF 文件的接口是 https://upload-pr4yueoqha-ue.a.run.app/to-to-anything/us-east1/upload
上传成功会返回
```json
{
  "type":"ok",
  "sourceInfo":{
    "sourceId":"9bc2PwgZwWJpXiYnJSDhB",
    "filename":"MaterialsStudio_8.0_InstallGuide.pdf",
    "displayName":"MaterialsStudio_8.0_InstallGuide.pdf",
    "status":"uploaded",
    "requiredCredits":{
      "credits":0,
      "pageCredits":61,
      "questionCredits":0,
      "numPdfs":0
    },
    "numPages":61,
    "uploadDate":1679893440261,
    "numBytes":708169
  }
}
```
此域名无法直接访问，也搜不到有效的的信息。猜测可能是 Google Cloud Run 的域名。是 chat_pdf 
作者自己开发的后端服务。
### 解析pdf

解析时，浏览器会请求 https://analyze-pr4yueoqha-ue.a.run.app/to-to-anything/us-east1/analyze
接口，若解析成功，会返回 {"success":true}

### 聊天交互

与 chat_pdf 交互的接口是 https://chat-pr4yueoqha-ue.a.run.app/to-to-anything/us-east1/chat
每次请求时，会携带完整的历史记录 history ，当前的用户输入是最新的一条 history 
记录。返回的是当前用户输入的消息的响应，包括：响应是否成功，用户的消息 ID，AI 的消息 ID，AI 的响应消息。
```json

{
  "type":"ok",
  "humanMsgId":"1679893669094",
  "aiMsgId":"1679893669095"
}
You can find information about installing Materials Studio on Linux systems on 
page 15 of this PDF file. It also provides additional steps that need to be 
taken if you are installing Materials Studio for use with a Linux Gaussian 
server, which can be found on page 21.

```

### chat pdf 分析总结

猜测 chat_pdf 的后端服务是由 Google Cloud Run 提供的，使用 google 接收上传的 PDF 文件，
使用 google 的 AI 服务进行解析，然后将解析的结果发给 chat_gpt。


解析完成后，由chat gpt 负责交互部分，将用户的输入转发给 chat gpt 通过在后台请求 open AI 的 API，
chat_gpt 会根据解析的结果 与用户的输入进行聊天交互。

---

## 当前版本实现

### 思路

核心功能：
- 调用 google-cloud-vision api，识别上传的 PDF 文件中的文字。
- 调用 open AI 的 API，将识别的文字作为 prompt，与chat gpt 开始一个会话。
- 将 chat gpt 的首次响应转发给用户。
- 将用户的后续输入转发给 chat gpt，将 chat gpt 的响应转发给用户。

后续可选功能：
- 将用户的会话记录保存到数据库中，方便用户查看历史记录。

### 步骤分解

1. （已实现）接受用户上传的 pdf 文件，将文件保存到某个目录下，将文件信息保存到数据库中。 
- 上传文件的接口：/upload
- 校验文件是否为 pdf 文件。
- 返回文件的绝对路径。

2. 调用 google-cloud-vision api，识别上传的 PDF 文件中的文字。

  - 需要创建一个 google cloud 项目，开通 vision api。
  - 确保当前的 google cloud 项目绑定了一个有效的付款结算方式。
  - 在 google cloud 项目中创建一个服务账号，下载 json 文件。
  - 将 json 文件放在项目根目录下，命名为 google_cloud_vision.json。
  - 安装 google-cloud-vision 库。
  - 在代码中调用 google-cloud-vision api，识别上传的 PDF 文件中的文字。

根据 google 的 API 手册，目前，PDF/TIFF 文档检测仅适用于存储在 Cloud Storage 
存储分区中的文件。响应 JSON 文件同样会保存到 Cloud Storage 存储分区。
https://cloud.google.com/vision/docs/pdf?hl=zh-cn

3. 调用 chat_gpt 的接口，将识别的文字作为 prompt，与chat gpt 开始一个会话。
open AI 的 API 文档：https://platform.openai.com/docs/api-reference/introduction

4. 将 chat gpt 的首次响应转发给用户。
5. 将用户的后续输入转发给 chat gpt，将 chat gpt 的响应转发给用户。

### 所需资源

- 可用的 google cloud 测试账号
- 可用的 open AI 测试账号
- 由于需要频繁与google cloud 和 open AI 通信，为了方便开发测试，
建议使用部署在外网的小型 VPS 服务器来开发。谷歌云主机、阿里云海外版、腾讯云海外版等都可以，基本的低配置就可以满足需求。
