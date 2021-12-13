# 基础规则

1. 使用单数的resource名，因为复数的变形规则不确定，会增加心智负担。
1. 只使用 GET/POST，大环境导致很多团队并不会适配这两个动词之外的其他动词，而且把行为根据动词来区分会导致难以通过路由判断行为。
1. 和原始的 Restful 倡导的一样，路径中只包含资源名。区别在于，我们允许（但不建议）最后出现一个动词或动词词组表达 API 功能，因为很多业务场景是纯粹的 HTTP 动词覆盖不到的，另一部分原因也是因为上述第 2 点提到的。
1. 设计 API URL 时候第一点考虑的是“这是一个什么资源，隶属于什么其他资源”，递归地问自己这个问题直至找到 DDD 中的聚合根。API 的参数应当属于业务条件（pageSize、pageNum）或实体属性（例如，根据商品类型过滤商品可以提取为type=...这一过滤条件）

# API Sample

## list

- method: `GET` 
- path: `/v1/resource/list`
- query:
   - pageNum {number}
   - pageSize {number}
   - orderBy { [ [<field>, <order>], ...] } - e.g. `[['updateTime', 'desc'], ['status', 'asc']]` 先按时间倒序排列，再按状态正序排列
   - search {string} - 搜索关键词
- body
- response
   - code
   - data
      - totalCount: {number}
      - pageNum: {number}
      - pageSize: {number}
      - resourceList: {Resource[]}

## getById

- method: `GET` 
- path: `/v1/resouce/:resourceId`
- param
   - resourceId
- reponse
   - code
   - data
      - resource: {Resource}


## create

- method: `POST`
- path: `/v1/resouce/`
- body
   - resource: {Resource}
- reponse
   - code
   - data
      - resource: {Resource}
## update


- method: `POST`
- path: `/v1/resouce/:resourceId`
- body
   - resource: {Resource}
- reponse
   - code
   - data
      - resource: {Resource}
## delete

- method: `POST`
- path: `/v1/resouce/:resourceId/delete`
- body
- reponse
   - code
   - data

# 原则

1. 关注点分离：接口应小而精，单一职责，做好一件事儿。例如：更新密码和修改个人资料虽然都是对person的update操作，但是应该拆解成两个接口，否则容易造成接口维护困难，容易混淆

# Ref

- [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) ✨✨✨✨✨ 强烈推荐，实操性很强，但自定义语法较多，需要根据情况重新设计
- [API设计的几条原则 - ThoughtWorks 少个分号](https://insights.thoughtworks.cn/how-to-design-api/) 
- [Google API 设计指南](https://cloud.google.com/apis/design) 
