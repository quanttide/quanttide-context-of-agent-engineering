# API 网关配置（传统网关 OpenAPI 签名调用）

> 2026-08 qtcloud-crowd 实证。FC 已挂系统级 API 网关（`api.example.com/<product>/...`，
> DNS CNAME → `<group>-cn-hangzhou.alicloudapi.com`）时，给新服务加网关路由的完整做法。

## 工具链结论

- **aliyun CLI 没有传统 API 网关产品命令**（`aliyun apigateway` 报 not valid；只有云原生 `apig`
  且需插件 `aliyun-cli-apig`——ListGateways 返回空说明网关不是云原生实例）
- **PyPI 无 apigateway 产品 SDK**（`alibabacloud-apigateway20180601` 不存在）；
  `alibabacloud_tea_openapi` 通用 SDK 有 py3.12/3.11 都触发的
  `quote() doesn't support 'encoding' for bytes` bug（Params 模型在
  `alibabacloud_tea_openapi.models`，不在 openapi_util）
- **可靠路径：Python 手写 RPC 签名**（标准库 hmac/hashlib/base64/urllib——零依赖）

## RPC 签名调用模式

```python
import json, hmac, hashlib, base64, time, uuid, urllib.parse, urllib.request

d = json.load(open('~/.aliyun/config.json'))
AK_ID = d['profiles'][0]['access_key_id']
AK_SEC = d['profiles'][0]['access_key_secret']
ENDPOINT = 'https://apigateway.cn-hangzhou.aliyuncs.com/'

def call(action, params=None):
    p = {
        'Action': action, 'Version': '2016-07-14', 'Format': 'JSON',
        'AccessKeyId': AK_ID, 'SignatureMethod': 'HMAC-SHA1',
        'SignatureVersion': '1.0', 'SignatureNonce': str(uuid.uuid4()),
        'Timestamp': time.strftime('%Y-%m-%dT%H:%M:%SZ', time.gmtime()),
        'RegionId': 'cn-hangzhou',
    }
    if params: p.update(params)
    keys = sorted(p.keys())
    qstr = '&'.join(f'{urllib.parse.quote(k, safe="")}={urllib.parse.quote(str(p[k]), safe="")}' for k in keys)
    string_to_sign = 'POST&%2F&' + urllib.parse.quote(qstr, safe='')
    sig = base64.b64encode(hmac.new((AK_SEC + '&').encode(), string_to_sign.encode(), hashlib.sha1).digest()).decode()
    p['Signature'] = sig
    req = urllib.request.Request(ENDPOINT, data=urllib.parse.urlencode(p).encode())
    with urllib.request.urlopen(req, timeout=30) as r:
        return json.loads(r.read())
```

## 配置流程（5 步）

1. **找组**：`DescribeApiGroups` → 找到组（量潮 qtcloud 组
   `34c138c4bec1405d942a57d9bb5ede37`，SubDomain 即 CNAME 目标）——响应在
   `ApiGroupAttributes.ApiGroupAttribute[]`
2. **列 API**：`DescribeApis`（GroupId + PageSize）→ `ApiSummarys.ApiSummary[]`
   （ApiId/ApiName）——量潮命名 `<product>-<resource>-<action>`
3. **拿模板**：`DescribeApi`（GroupId + ApiId——选带 `{id}` 路径参数的 API 作模板，
   如 task-claim）→ 取 ServiceConfig（ServiceProtocol=HTTP / ServiceAddress=FC URL /
   ServicePath / ServiceHttpMethod）与 RequestConfig（RequestPath / Method / BodyFormat）
   ——注意响应可能没有 body 包装（用 `.get('body', resp)` 兜底）
4. **创建**：`CreateApi`——改 ApiName、RequestConfig.RequestPath（`/qtcrowd/api/tasks/{id}/claim`）、
   ServiceConfig.ServicePath + ServiceAddress（新 FC URL：`<fn>-<rand>.cn-hangzhou.fcapp.run`）、
   ServiceHttpMethod；**路径参数 API 必须带 RequestParameters/ServiceParameters**
   （`json.dumps` 数组字符串——从模板原样复制）；AuthType=ANONYMOUS、Visibility=PUBLIC、
   ResultType=JSON、ResultSample='{}'
5. **发布**：`DeployApi`（GroupId + ApiId + StageName=RELEASE + Description）——未 Deploy 的
   API 线上 404；然后 `curl https://api.example.com/<product>/api/tasks` 验证

## 要点

- **CreateApi 的参数是 JSON 字符串**（RequestConfig/ServiceConfig/RequestParameters 等都要
  `json.dumps`），不是嵌套结构
- 从模板复制时只改需要变的部分（FC URL/路径/名字），ServiceConfig 其余字段保持
- 惯例：terraform `outputs.tf` 记录 `apigateway_domain` + `apigateway_apis` 清单
  （网关手动配——IaC 只留审计记录，与 qtcloud-crowd 对齐）
- FC URL 获取：`aliyun fc ListFunctions`（函数名 `<product>-prod`）+
  `aliyun fc ListTriggers --functionName <fn>` → `httpTrigger.urlInternet`
- 验证顺序：FC 直连 health 200 → 网关路由 200 → 全链路（网关→FC→数据）
