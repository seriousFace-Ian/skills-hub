# API 文档示例

## Example (Group by route name)

When multiple APIs share a route prefix, start with the summary table, then detail each one.

**Group by route name**：`/api/v1/user`

| alias | method | url | Ready | description |
|------ |------|----------|----------|------|
| getUserList | GET | /api/v1/user/list | done | 获取用户列表，支持分页和关键词搜索 |
| createUser | POST | /api/v1/user/create | in progress | 创建新用户 |
| deleteUser | DELETE | /api/v1/user/{id} | not started | 删除指定用户 |

---

### getUserList

**Description**: Retrieve a list of users, supporting pagination and keyword search.

**URL**：`/api/v1/user/list`

**Request Method**：GET

#### Request Params

| param | type | required | description |
|--------|------|------|------|
| pageNum | Integer | 是 | 页码，从1开始 |
| pageSize | Integer | 是 | 每页条数，最大100 |
| keyword | String | 否 | 搜索关键词（用户名或手机号） |
| status | Integer | 否 | 用户状态：0-禁用，1-启用 |

#### Response Result

| param | type | description |
|--------|------|------|
| code | Integer | 状态码，200表示成功 |
| msg | String | 提示信息 |
| data | ResponseData | 返回数据 |

**ResponseData**
| param | type | description |
|--------|------|------|
| total | Integer | 总条数 |
| list | Array<User> | 用户列表 |

**User**
| param | type | description |
|--------|------|------|
| userId | String | 用户ID |
| userName | String | 用户名 |
| phone | String | 手机号（脱敏） |
| status | Integer | 用户状态：0-禁用，1-启用 |
| createTime | String | 创建时间，格式 yyyy-MM-dd HH:mm:ss |

#### Request example

**GetUserListRequest**

```json
{
  "pageNum": 1,
  "pageSize": 10,
  "name": "张三",
}
```

#### Response example

**GetUserListResponse**

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "total": 2,
    "list": [
      {
        "userId": "u001",
        "userName": "张三",
        "phone": "138****0001",
        "status": 1,
        "createTime": "2024-01-15 09:30:00"
      },
      {
        "userId": "u002",
        "userName": "张三丰",
        "phone": "139****0002",
        "status": 1,
        "createTime": "2024-02-20 14:00:00"
      }
    ]
  }
}
```

---

*(createUser and deleteUser details would follow in the same pattern)*

---

## 示例二：单个 API（无路由分组）

Skip the summary table entirely and go directly to the API detail section.

---

## Notes

- Two response examples are shown because success and error have different `data` shapes.
- Nested array fields use `data.list[].fieldName` notation in the table.
- Phone numbers are shown masked as they would appear from the API.
- If there were a third response shape (e.g. permission denied), it would be documented only in the table, not as a third JSON block.
- The route group heading (`**路由分组**`) is optional; include it if the user specifies a shared prefix.
