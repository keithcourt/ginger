
# 流程引擎设计
## 一、组织机构配置

1.每个应用需要配置获取组织机构的接口：
* 按部门分类  
    
```
    部门id、部门名称、上级部门id 
        人员id、姓名、手机、邮箱
    
    数据结构：
    {
        dep_id,
        dep_name,
        parent_dep_id,
        members: [{
            id,
            name,
            phone,
            mail,
            isManager
        }],
        sub_dep: [
            ...
        ]
    }
```

* 按角色分类  

```
    角色id、角色名称
        人员id、姓名、手机、邮箱
        
    数据结构：
    {
        role_id,
        role_name,
        members: [{
            id,
            name,
            phone,
            mail,
            isManager
        }]
    }
```

## 二、新建流程模版

1. 新建流程模版时，需要选择一个表单关联该流程模版
2. 表单列表从其他系统中获取
    
## 三、流程设定
#### 1. 画流程图，建立节点和流程的流转关系
* ###### 节点分类
  开始节点、结束节点、业务节点、子流程节点
* ###### 连接线分类
  并行连接、条件连接
#### 2. 给节点配置属性
* ###### 节点名称
    通过 `节点属性` 修改当前节点名称
* ###### 负责人
    通过 `节点属性` > `基础属性` > `负责人` 便可以选择指定的负责人从配置的接口中获取组织机构人员（部门；角色；成员；动态负责[流程发起人、流程发起人部门主管]）
* ###### 操作权限         
    通过 `节点属性` >  `基础属性` > `操作权限` 对该节点负责人在处理待办数据时的操作权限规定。决定了字段是否可见，是否可编辑，以及简报字段
    * 可见：即节点负责人对该字段是否可见。勾选，则该节点负责人可见这个字。 不勾选则不可见。  
    * 可编辑：即节点负责人对该字段的值是否可编辑。勾选，则该节点负责人对该字段的值可编辑。不勾选则不可编辑。
    * 流程简报：即流程的简报信息，在该节点负责人的「流程消息提醒」和「流程管理」中，会显示选择的字段，作为简报信息
* ###### 节点操作设置 
    通过 `节点属性` >  `更多属性` > `节点操作` 配置节点负责人可以进行的操作，可选操作有以下6个：**提交；暂存；提交并打印；回退；转交；结束流程**。其中「提交」为必选操作，不可关闭。其他的5个操作均为自选操作，可开启或关闭
    
    操作类型 | 功能 
    ---|---
    提交 | 保存在此节点中的操作，将数据流转到下一节点中 
    暂存 | 保存在此节点中的操作，并且流程数据保留在我的待办中 
    提交并打印 | 保存在此节点中的操作，将数据流转到下一节点中，同时进入打印的界面进行数据打印  
    回退 | 保存在此节点中的操作，并且流程数据退回到某个节点 
    转交 | 将该条待办数据转交给其他成员进行处理 
    结束流程| 保存在此节点中的操作，并且将流程直接结束掉，不再往下流转 |

* ###### 流转规则
    * 单个节点多个任务：通过 `节点属性` >  `更多属性` > `流转规则` 设置
        `本条规则适用于一个节点内的判断，当这个节点内的负责人有多个时的场景`
        * 任意负责人提交后进入下一节点：`即该节点内只要一个负责人点击提交，则数据进入下一节点`
        * 所有负责人提交后进入下一节点：`即该节点内需要所有负责人都点击提交，数据才会进入下一节点`
        有一人点击回退，则数据回退至上一节点。

    * 节点间，一个节点流转到多个分支：`数据会同时进入所有满足条件的节点` 
        * 如果是并行节点，直接进入下一节点
        * 如果是条件节点，根据判断是否进入下一节点
        
    * 节点间，多个分支流转到共同子节点：`查找所有的父节点，所有父节点全部完成后，数据流转入该共同子节点`
    
    * 回退：在节点操作回退按钮中配置
        * `编辑回退` > `允许回退到` > `指定范围内节点` > 勾选该节点上方的节点，作为回退时的可选范围
        * `编辑回退` > `允许回退到` > `上一节点` > 则点击回退，直接退回到上一节点


## 三、应用系统使用流程

1. 应用首先取得流程服务的授权
2. 管理员进入流程的管理页面后，设置组织机构接口、业务分类
3. 创建流程、关联表单、属性配置
4. 发起流程
    * 应用系统根据 `template_id` 请求流程引擎发起流程
    * 流程创建实例并创建开始节点的待办任务
5. 流程管理：
    * 我的待办任务
  
    ```
    接口：
    
    请求参数：{
        tenant_id:
        app_id:
        user_id;
    },
    
    返回参数：
    list：[{
        
        task_id: 
        task_name:
        task_type:
        start_time:
        start_user:
        instance_id:
        instance_name:
        ...
        
        "properties":{
            "btnOprate":[
                {"name":"提交", "isOpen":false, "rowkey":"1"},
                {"name":"暂存","isOpen":false,"rowkey":"2"},
                {"name":"提交并打印","isOpen":false,"rowkey":"3"},
                {"name":"结束流程","isOpen":false,"rowkey":"4"}
             ],
            "feildAuth":[
                {"id":"dsfdsfdsgfd1232434","controlName":"收款人","isVisable":true,"notice":false,"isEdit":false},
                {"id":"jkyruktdyfj1564543","controlName":"收款金额","isVisable":true,"notice":false,"isEdit":false},
                {"id":"dsfvmjuikuyd123243","controlName":"收款时间","isVisable":true,"notice":false,"isEdit":false},
                {"id":"moiwesgmhjdsgfd123","controlName":"付款人","isVisable":true,"notice":false,"isEdit":false}
            ]
        },
        
    },
        
    ...
    ]
    ```

    * 我发起的
    * 我参与的
    
    ```
    接口：
    
    请求参数：{
        tenant_id:
        app_id:
        user_id;
    },
   
    返回参数：我发起的流程信息
    list：[{
        
        instance_id: 
        instance_name:
        task_id:
        task_status:
        task_name:
        
       },
        ...
    ]
    ```
6. 节点操作
   * 提交
       
    ```
    接口：
    
    请求参数：{
        task_id:
        user_id:
        parameters: {
            field1: value1,
            field2: value2
        }
    },
    
    返回参数：
    result: {
        success: true,
        msg: "",
    }
   ```
   
   * 结束
   
   ```
    接口：
    
    请求参数：{
        instance_id:
        user_id:
    },
    
    返回参数：
    result: {
        success: true,
        msg: "",
    }
  ```
  


