# 20240702

> 记一次Elasticsearch折叠查询后分页精度丢失的问题

使用的ES版本:

```json
{
    "name": "dev-es1-172.16.74.48",
    "cluster_name": "dev-es1",
    "cluster_uuid": "uY4ZpL0JRAWL_ulE39QpOg",
    "version": {
        "number": "7.6.2",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
        "build_date": "2020-03-26T06:34:37.794943Z",
        "build_snapshot": false,
        "lucene_version": "8.4.0",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```

折叠+分页查询DSL：

```json
{
    "from": 0,
    "size": 10, //查询10条数据
  // 排序
    "sort": [
        {
            "updateTime": "desc"
        }
    ],
  // 查询条件
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "requestBody.bizId.keyword": "cscec202304170000020455"
                    }
                }
            ]
        }
    },
  // 折叠字段
    "collapse": {
        "field": "requestBody.biz.keyword",
      // 折叠条数内部排序
        "inner_hits": {
            "name": "sorted",
            "sort": [
                {
                    "updateTime": "desc"
                }
            ],
          // 折叠内部排序后取一条数据
            "size": 1
        }
    },
    "aggs": {
      // 使用聚合查询(命名为biz_count)得到每一条折叠数据被折叠的条数
        "biz_count": {
            "terms": {
                "field": "requestBody.biz.keyword"
            }
        }
    }
}
```

查询结果：

```json
{
    "took": 17,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
          // 这里是一共命中8条数据，但是折叠后只有2条
            "value": 8,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "wf_process_instance",
                "_type": "_doc",
                "_id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                "_score": null,
                "_source": {
                    "processStatus": 2,
                    "fromUserId": 100001,
                    "completeTime": 1681723283222,
                    "assigneeInfos": "[]",
                    "updateTime": 1683637656983,
                    "title": "【zzz三局下级】【测试企业】标书发布审批",
                    "handleUserIds": [],
                    "flowName": "PUBLISH_TENDER_AUDIT",
                    "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                    "processDefKey": "cn.yzw.cn.audit.14094",
                    "form": [],
                    "requestBody": {
                        "biz": "PUBLISH_TENDER_AUDIT+cscec202304170000020455",
                        "orgName": "测试企业",
                        "requestType": 0,
                        "mobileTerminal": true,
                        "orgCode": "0001010090030001",
                        "bizDomainId": 1006,
                        "bizId": "cscec202304170000020455",
                        "tenantId": "cscec",
                        "auditNodeCode": "PUBLISH_TENDER_AUDIT",
                        "applicationId": "522011",
                        "isRevoke": 1,
                        "auditNodeName": "标书发布审批"
                    },
                    "instanceNo": "CC1681723038485349010",
                    "bizId": "cscec202304170000020455",
                    "startTime": 1681723039939,
                    "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                    "id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                    "handledUserIds": [
                        10344002
                    ]
                },
                "fields": {
                    "requestBody.biz.keyword": [
                        "PUBLISH_TENDER_AUDIT+cscec202304170000020455"
                    ]
                },
                "sort": [
                    1683637656983
                ],
                "inner_hits": {
                    "sorted": {
                        "hits": {
                            "total": {
                                "value": 3,
                                "relation": "eq"
                            },
                            "max_score": null,
                            "hits": [
                                {
                                    "_index": "wf_process_instance",
                                    "_type": "_doc",
                                    "_id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                                    "_score": null,
                                    "_source": {
                                        "processStatus": 2,
                                        "fromUserId": 100001,
                                        "completeTime": 1681723283222,
                                        "assigneeInfos": "[]",
                                        "updateTime": 1683637656983,
                                        "title": "【zzz三局下级】【测试企业】标书发布审批",
                                        "handleUserIds": [],
                                        "flowName": "PUBLISH_TENDER_AUDIT",
                                        "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                                        "processDefKey": "cn.yzw.cn.audit.14094",
                                        "form": [],
                                        "requestBody": {
                                            "biz": "PUBLISH_TENDER_AUDIT+cscec202304170000020455",
                                            "orgName": "测试企业",
                                            "requestType": 0,
                                            "mobileTerminal": true,
                                            "orgCode": "0001010090030001",
                                            "bizDomainId": 1006,
                                            "bizId": "cscec202304170000020455",
                                            "tenantId": "cscec",
                                            "auditNodeCode": "PUBLISH_TENDER_AUDIT",
                                            "applicationId": "522011",
                                            "isRevoke": 1,
                                            "auditNodeName": "标书发布审批"
                                        },
                                        "instanceNo": "CC1681723038485349010",
                                        "bizId": "cscec202304170000020455",
                                        "startTime": 1681723039939,
                                        "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                                        "id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                                        "handledUserIds": [
                                            10344002
                                        ]
                                    },
                                    "sort": [
                                        1683637656983
                                    ]
                                }
                            ]
                        }
                    }
                }
            },
            {
                "_index": "wf_process_instance",
                "_type": "_doc",
                "_id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                "_score": null,
                "_source": {
                    "processStatus": 2,
                    "fromUserId": 100001,
                    "completeTime": 1681722297746,
                    "assigneeInfos": "[{\"userList\":[{\"orgName\":\"测试企业\",\"position\":\"12\",\"userName\":\"draven2\",\"userId\":\"315342\"}],\"level\":1,\"nodeKey\":\"Activity_AuditNode_45787\"}]",
                    "updateTime": 1683637656504,
                    "title": "【zzz三局下级】【测试企业】联采管控审批",
                    "handleUserIds": [],
                    "flowName": "联采管控-继续招标-测试审批流",
                    "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                    "processDefKey": "cn.yzw.cn.audit.14457",
                    "form": [],
                    "requestBody": {
                        "biz": "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455",
                        "orgName": "测试企业",
                        "requestType": 0,
                        "orgCode": "0001010090030001",
                        "bizDomainId": 1006,
                        "bizId": "cscec202304170000020455",
                        "tenantId": "cscec",
                        "auditNodeCode": "REGIONAL_PURCHASE_AUDIT",
                        "applicationId": "522011",
                        "isRevoke": 1,
                        "auditNodeName": "联采管控审批"
                    },
                    "instanceNo": "CC1681722274410650985",
                    "bizId": "cscec202304170000020455",
                    "startTime": 1681722275413,
                    "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                    "id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                    "handledUserIds": [
                        315342
                    ]
                },
                "fields": {
                    "requestBody.biz.keyword": [
                        "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455"
                    ]
                },
                "sort": [
                    1683637656504
                ],
                "inner_hits": {
                    "sorted": {
                        "hits": {
                            "total": {
                                "value": 5,
                                "relation": "eq"
                            },
                            "max_score": null,
                            "hits": [
                                {
                                    "_index": "wf_process_instance",
                                    "_type": "_doc",
                                    "_id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                                    "_score": null,
                                    "_source": {
                                        "processStatus": 2,
                                        "fromUserId": 100001,
                                        "completeTime": 1681722297746,
                                        "assigneeInfos": "[{\"userList\":[{\"orgName\":\"测试企业\",\"position\":\"12\",\"userName\":\"draven2\",\"userId\":\"315342\"}],\"level\":1,\"nodeKey\":\"Activity_AuditNode_45787\"}]",
                                        "updateTime": 1683637656504,
                                        "title": "【zzz三局下级】【测试企业】联采管控审批",
                                        "handleUserIds": [],
                                        "flowName": "联采管控-继续招标-测试审批流",
                                        "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                                        "processDefKey": "cn.yzw.cn.audit.14457",
                                        "form": [],
                                        "requestBody": {
                                            "biz": "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455",
                                            "orgName": "测试企业",
                                            "requestType": 0,
                                            "orgCode": "0001010090030001",
                                            "bizDomainId": 1006,
                                            "bizId": "cscec202304170000020455",
                                            "tenantId": "cscec",
                                            "auditNodeCode": "REGIONAL_PURCHASE_AUDIT",
                                            "applicationId": "522011",
                                            "isRevoke": 1,
                                            "auditNodeName": "联采管控审批"
                                        },
                                        "instanceNo": "CC1681722274410650985",
                                        "bizId": "cscec202304170000020455",
                                        "startTime": 1681722275413,
                                        "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                                        "id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                                        "handledUserIds": [
                                            315342
                                        ]
                                    },
                                    "sort": [
                                        1683637656504
                                    ]
                                }
                            ]
                        }
                    }
                }
            }
        ]
    },
  // 聚合后折叠的数量
    "aggregations": {
        "biz_count": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
          // 返回2条折叠数据，分别折叠了5条和3条数据
            "buckets": [
                {
                    "key": "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455",
                    "doc_count": 5
                },
                {
                    "key": "PUBLISH_TENDER_AUDIT+cscec202304170000020455",
                    "doc_count": 3
                }
            ]
        }
    }
}
```

上面的查询结果只返回的hit命中数量，平时我们都是使用命中数量作为总数，但是在使用折叠查询后使用hit数量作为总数就会导致分页精度丢失，出现分页数大于实际页数的问题。我的解决方案是使用cardinality查询获取去重后的数量用于分页处理。

修改后的查询DSL：

```json
{
    "from": 0,
    "size": 10, //查询10条数据
  // 排序
    "sort": [
        {
            "updateTime": "desc"
        }
    ],
  // 查询条件
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "requestBody.bizId.keyword": "cscec202304170000020455"
                    }
                }
            ]
        }
    },
  // 折叠字段
    "collapse": {
        "field": "requestBody.biz.keyword",
      // 折叠条数内部排序
        "inner_hits": {
            "name": "sorted",
            "sort": [
                {
                    "updateTime": "desc"
                }
            ],
          // 折叠内部排序后取一条数据
            "size": 1
        }
    },
    "aggs": {
      // 使用聚合查询(命名为biz_count)得到每一条折叠数据被折叠的条数
        "biz_count": {
            "terms": {
                "field": "requestBody.biz.keyword"
            }
        },
      // 增加cardinality(命名为data_total)去重得到折叠后的总数
        "data_total": {
            "cardinality": {
                "field": "requestBody.biz.keyword"
            }
        }
    }
}
```

修改后的查询结果：

```json
{
    "took": 17,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
          // 这里是一共命中8条数据，但是折叠后只有2条
            "value": 8,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "wf_process_instance",
                "_type": "_doc",
                "_id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                "_score": null,
                "_source": {
                    "processStatus": 2,
                    "fromUserId": 100001,
                    "completeTime": 1681723283222,
                    "assigneeInfos": "[]",
                    "updateTime": 1683637656983,
                    "title": "【zzz三局下级】【测试企业】标书发布审批",
                    "handleUserIds": [],
                    "flowName": "PUBLISH_TENDER_AUDIT",
                    "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                    "processDefKey": "cn.yzw.cn.audit.14094",
                    "form": [],
                    "requestBody": {
                        "biz": "PUBLISH_TENDER_AUDIT+cscec202304170000020455",
                        "orgName": "测试企业",
                        "requestType": 0,
                        "mobileTerminal": true,
                        "orgCode": "0001010090030001",
                        "bizDomainId": 1006,
                        "bizId": "cscec202304170000020455",
                        "tenantId": "cscec",
                        "auditNodeCode": "PUBLISH_TENDER_AUDIT",
                        "applicationId": "522011",
                        "isRevoke": 1,
                        "auditNodeName": "标书发布审批"
                    },
                    "instanceNo": "CC1681723038485349010",
                    "bizId": "cscec202304170000020455",
                    "startTime": 1681723039939,
                    "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                    "id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                    "handledUserIds": [
                        10344002
                    ]
                },
                "fields": {
                    "requestBody.biz.keyword": [
                        "PUBLISH_TENDER_AUDIT+cscec202304170000020455"
                    ]
                },
                "sort": [
                    1683637656983
                ],
                "inner_hits": {
                    "sorted": {
                        "hits": {
                            "total": {
                                "value": 3,
                                "relation": "eq"
                            },
                            "max_score": null,
                            "hits": [
                                {
                                    "_index": "wf_process_instance",
                                    "_type": "_doc",
                                    "_id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                                    "_score": null,
                                    "_source": {
                                        "processStatus": 2,
                                        "fromUserId": 100001,
                                        "completeTime": 1681723283222,
                                        "assigneeInfos": "[]",
                                        "updateTime": 1683637656983,
                                        "title": "【zzz三局下级】【测试企业】标书发布审批",
                                        "handleUserIds": [],
                                        "flowName": "PUBLISH_TENDER_AUDIT",
                                        "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                                        "processDefKey": "cn.yzw.cn.audit.14094",
                                        "form": [],
                                        "requestBody": {
                                            "biz": "PUBLISH_TENDER_AUDIT+cscec202304170000020455",
                                            "orgName": "测试企业",
                                            "requestType": 0,
                                            "mobileTerminal": true,
                                            "orgCode": "0001010090030001",
                                            "bizDomainId": 1006,
                                            "bizId": "cscec202304170000020455",
                                            "tenantId": "cscec",
                                            "auditNodeCode": "PUBLISH_TENDER_AUDIT",
                                            "applicationId": "522011",
                                            "isRevoke": 1,
                                            "auditNodeName": "标书发布审批"
                                        },
                                        "instanceNo": "CC1681723038485349010",
                                        "bizId": "cscec202304170000020455",
                                        "startTime": 1681723039939,
                                        "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                                        "id": "a6b43687-dd00-11ed-a24c-76f43a5988a5",
                                        "handledUserIds": [
                                            10344002
                                        ]
                                    },
                                    "sort": [
                                        1683637656983
                                    ]
                                }
                            ]
                        }
                    }
                }
            },
            {
                "_index": "wf_process_instance",
                "_type": "_doc",
                "_id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                "_score": null,
                "_source": {
                    "processStatus": 2,
                    "fromUserId": 100001,
                    "completeTime": 1681722297746,
                    "assigneeInfos": "[{\"userList\":[{\"orgName\":\"测试企业\",\"position\":\"12\",\"userName\":\"draven2\",\"userId\":\"315342\"}],\"level\":1,\"nodeKey\":\"Activity_AuditNode_45787\"}]",
                    "updateTime": 1683637656504,
                    "title": "【zzz三局下级】【测试企业】联采管控审批",
                    "handleUserIds": [],
                    "flowName": "联采管控-继续招标-测试审批流",
                    "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                    "processDefKey": "cn.yzw.cn.audit.14457",
                    "form": [],
                    "requestBody": {
                        "biz": "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455",
                        "orgName": "测试企业",
                        "requestType": 0,
                        "orgCode": "0001010090030001",
                        "bizDomainId": 1006,
                        "bizId": "cscec202304170000020455",
                        "tenantId": "cscec",
                        "auditNodeCode": "REGIONAL_PURCHASE_AUDIT",
                        "applicationId": "522011",
                        "isRevoke": 1,
                        "auditNodeName": "联采管控审批"
                    },
                    "instanceNo": "CC1681722274410650985",
                    "bizId": "cscec202304170000020455",
                    "startTime": 1681722275413,
                    "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                    "id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                    "handledUserIds": [
                        315342
                    ]
                },
                "fields": {
                    "requestBody.biz.keyword": [
                        "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455"
                    ]
                },
                "sort": [
                    1683637656504
                ],
                "inner_hits": {
                    "sorted": {
                        "hits": {
                            "total": {
                                "value": 5,
                                "relation": "eq"
                            },
                            "max_score": null,
                            "hits": [
                                {
                                    "_index": "wf_process_instance",
                                    "_type": "_doc",
                                    "_id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                                    "_score": null,
                                    "_source": {
                                        "processStatus": 2,
                                        "fromUserId": 100001,
                                        "completeTime": 1681722297746,
                                        "assigneeInfos": "[{\"userList\":[{\"orgName\":\"测试企业\",\"position\":\"12\",\"userName\":\"draven2\",\"userId\":\"315342\"}],\"level\":1,\"nodeKey\":\"Activity_AuditNode_45787\"}]",
                                        "updateTime": 1683637656504,
                                        "title": "【zzz三局下级】【测试企业】联采管控审批",
                                        "handleUserIds": [],
                                        "flowName": "联采管控-继续招标-测试审批流",
                                        "fromUserName": "{\"account\":\"jcadmin\",\"appId\":\"3009_qa\",\"id\":100001,\"name\":\"呵呵1\",\"userSystem\":\"auac\"}",
                                        "processDefKey": "cn.yzw.cn.audit.14457",
                                        "form": [],
                                        "requestBody": {
                                            "biz": "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455",
                                            "orgName": "测试企业",
                                            "requestType": 0,
                                            "orgCode": "0001010090030001",
                                            "bizDomainId": 1006,
                                            "bizId": "cscec202304170000020455",
                                            "tenantId": "cscec",
                                            "auditNodeCode": "REGIONAL_PURCHASE_AUDIT",
                                            "applicationId": "522011",
                                            "isRevoke": 1,
                                            "auditNodeName": "联采管控审批"
                                        },
                                        "instanceNo": "CC1681722274410650985",
                                        "bizId": "cscec202304170000020455",
                                        "startTime": 1681722275413,
                                        "_class": "cn.yzw.iec.workflow.server.common.model.param.entity.ProcessEsInstance",
                                        "id": "df47774d-dcfe-11ed-a24c-76f43a5988a5",
                                        "handledUserIds": [
                                            315342
                                        ]
                                    },
                                    "sort": [
                                        1683637656504
                                    ]
                                }
                            ]
                        }
                    }
                }
            }
        ]
    },
  // 聚合后折叠的数量
    "aggregations": {
      // 去重后的总数为2，用这里的总数做分页即可
        "data_total": {
            "value": 2
        },
        "biz_count": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
          // 返回2条折叠数据，分别折叠了5条和3条数据
            "buckets": [
                {
                    "key": "REGIONAL_PURCHASE_AUDIT+cscec202304170000020455",
                    "doc_count": 5
                },
                {
                    "key": "PUBLISH_TENDER_AUDIT+cscec202304170000020455",
                    "doc_count": 3
                }
            ]
        }
    }
}
```