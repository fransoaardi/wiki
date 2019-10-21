# Github Enterprise

## enterprise version 확인

```bash
curl http(s)://hostname/api/v3/
```

response로 "documentation_url"에 github enterprise version 정보를 알 수 있다.

```json
{
  "message": "Must authenticate to access this API.",
  "documentation_url": "https://developer.github.com/enterprise/2.12/v3"
}
```
