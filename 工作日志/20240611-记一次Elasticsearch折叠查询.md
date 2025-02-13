# 20240611

> 记一次Elasticsearch折叠查询，需求是根据索引中的2个字段进行折叠(collapse)查询，使用的Elasticsearch版本是7.6.2

### 1个字段折叠查询：

> aggs查询是为了统计折叠的数量，存在性能问题
>
> inner_hits是为了被折叠的数据局部排序取第一条

```json
{
    "from": 0,
    "size": 2,
    "sort": [
        {
            "updateTime": "desc"
        }
    ],
    "query": {
        "match_phrase": {
            "requestBody.bizId.keyword": "402096417704790016"
        }
    },
    "collapse": {
        "field": "requestBody.bizId.keyword",
        "inner_hits": {
            "name": "sorted",
            "sort": [
                {
                    "updateTime": "desc"
                }
            ],
            "size": 1
        }
    },
    "aggs": {
        "biz_count": {
            "terms": {
                "field": "requestBody.bizId.keyword"
            }
        }
    }
}
```

响应结果：

```json
{
    "took": 15,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 31,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "wf_process_instance",
                "_type": "_doc",
                "_id": "4f93b277-b431-11ed-8d05-0edcbd4615db",
                "_score": null,
                "_source": {
                    "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                    "id": "4f93b277-b431-11ed-8d05-0edcbd4615db",
                    "title": "招标执行-物资-无清单",
                    "form": [],
                    "fromUserId": 100001,
                    "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵\",\"userSystem\":\"auac\"}",
                    "handleUserIds": [],
                    "startTime": 1677235940641,
                    "updateTime": 1683636798250,
                    "completeTime": 1677235984716,
                    "handledUserIds": [
                        10302007
                    ],
                    "processDefKey": "cn.yzw.cn.audit.14130",
                    "processStatus": 3,
                    "assigneeInfos": "[{\"userList\":[{\"orgName\":\"zyp审批中心测试上级组织（勿动）\",\"userName\":\"张奉先\",\"userId\":\"10302007\"}],\"level\":1,\"nodeKey\":\"Activity_AuditNode_43720\"}]",
                    "requestBody": {
                        "bizId": "402096417704790016",
                        "requestType": 0,
                        "orgCode": "00010100",
                        "orgName": "中国建筑股份有限公司",
                        "auditNodeCode": "contract_center_template",
                        "auditNodeName": "合同模版审批",
                        "bizDomainId": 1006,
                        "applicationId": "522011",
                        "tenantId": "cscec",
                        "isRevoke": 1,
                        "mobileTerminal": true
                    },
                    "bizId": "402096417704790016",
                    "flowName": "合同模板审批流测试",
                    "instanceNo": "CC1677235939948214020"
                },
                "fields": {
                    "requestBody.bizId.keyword": [
                        "402096417704790016"
                    ]
                },
                "sort": [
                    1683636798250
                ],
                "inner_hits": {
                    "sorted": {
                        "hits": {
                            "total": {
                                "value": 31,
                                "relation": "eq"
                            },
                            "max_score": null,
                            "hits": [
                                {
                                    "_index": "wf_process_instance",
                                    "_type": "_doc",
                                    "_id": "4f93b277-b431-11ed-8d05-0edcbd4615db",
                                    "_score": null,
                                    "_source": {
                                        "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                                        "id": "4f93b277-b431-11ed-8d05-0edcbd4615db",
                                        "title": "招标执行-物资-无清单",
                                        "form": [],
                                        "fromUserId": 100001,
                                        "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵\",\"userSystem\":\"auac\"}",
                                        "handleUserIds": [],
                                        "startTime": 1677235940641,
                                        "updateTime": 1683636798250,
                                        "completeTime": 1677235984716,
                                        "handledUserIds": [
                                            10302007
                                        ],
                                        "processDefKey": "cn.yzw.cn.audit.14130",
                                        "processStatus": 3,
                                        "assigneeInfos": "[{\"userList\":[{\"orgName\":\"zyp审批中心测试上级组织（勿动）\",\"userName\":\"张奉先\",\"userId\":\"10302007\"}],\"level\":1,\"nodeKey\":\"Activity_AuditNode_43720\"}]",
                                        "requestBody": {
                                            "bizId": "402096417704790016",
                                            "requestType": 0,
                                            "orgCode": "00010100",
                                            "orgName": "中国建筑股份有限公司",
                                            "auditNodeCode": "contract_center_template",
                                            "auditNodeName": "合同模版审批",
                                            "bizDomainId": 1006,
                                            "applicationId": "522011",
                                            "tenantId": "cscec",
                                            "isRevoke": 1,
                                            "mobileTerminal": true
                                        },
                                        "bizId": "402096417704790016",
                                        "flowName": "合同模板审批流测试",
                                        "instanceNo": "CC1677235939948214020"
                                    },
                                    "sort": [
                                        1683636798250
                                    ]
                                }
                            ]
                        }
                    }
                }
            }
        ]
    },
    "aggregations": {
        "biz_count": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": "402096417704790016",
                    "doc_count": 31
                }
            ]
        }
    }
}
```

### 问题：collapse不支持多个独立字段的折叠查询！！！

- 方法一：使用runtime_mappings在查询是动态添加字段

```jso
{
    "from": 0,
    "size": 2,
    "sort": [
        {
            "updateTime": "desc"
        }
    ],
    "query": {
        "match_phrase": {
            "requestBody.bizId.keyword": "402096417704790016"
        }
    },
    "runtime_mappings": {
        "my_biz": {
            "type": "keyword",
            "script": "doc['requestBody.bizId.keyword'].value + '+' + doc['requestBody.auditNodeCode.keyword'].value"
        }
    },
    "collapse": {
        "field": "requestBody.bizId.keyword",
        "inner_hits": {
            "name": "sorted",
            "sort": [
                {
                    "updateTime": "desc"
                }
            ],
            "size": 1
        }
    },
    "aggs": {
        "biz_count": {
            "terms": {
                "field": "requestBody.bizId.keyword"
            }
        }
    }
}
```

响应结果：

```json
{
    "error": {
        "root_cause": [
            {
                "type": "parsing_exception",
                "reason": "Unknown key for a START_OBJECT in [runtime_mappings].",
                "line": 14,
                "col": 25
            }
        ],
        "type": "parsing_exception",
        "reason": "Unknown key for a START_OBJECT in [runtime_mappings].",
        "line": 14,
        "col": 25
    },
    "status": 400
}
```

runtime_mappings是elasticsearch 7.7之后提出的功能，7.6.2不支持！！！

- 方法二：创建一个新的字段(Field)，更新历史数据