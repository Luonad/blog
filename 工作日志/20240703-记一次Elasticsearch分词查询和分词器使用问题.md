# 20240703

> 记一次Elasticsearch分词查询和分词器使用问题

maping如下：

```json
{
	"wf_process_instance": {
		"mappings": {
			"properties": {
				"_class": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"assigneeInfos": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"batchEnable": {
					"type": "boolean"
				},
				"bizId": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"completeTime": {
					"type": "long"
				},
				"doc": {
					"properties": {
						"title": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						}
					}
				},
				"flowName": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"form": {
					"properties": {
						"key": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"value": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						}
					}
				},
				"fromUserId": {
					"type": "long"
				},
				"fromUserName": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"handleUserIds": {
					"properties": {
						"userId": {
							"type": "long"
						},
						"userName": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						}
					}
				},
				"handledUserIds": {
					"type": "long"
				},
				"id": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"instanceCcUserList": {
					"properties": {
						"userId": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"userName": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						}
					}
				},
				"instanceNo": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"processDefKey": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"processStatus": {
					"type": "long"
				},
				"requestBody": {
					"properties": {
						"RS-WEB-HOST": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"applicationId": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"auditCount": {
							"type": "long"
						},
						"auditEasyNodeId": {
							"type": "long"
						},
						"auditEasyOrgCode": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"auditNodeCode": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"auditNodeName": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"batchEnable": {
							"type": "boolean"
						},
						"biz": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"bizDomainId": {
							"type": "long"
						},
						"bizId": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"isRevoke": {
							"type": "long"
						},
						"mobileTerminal": {
							"type": "boolean"
						},
						"mro_domain": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"orgCode": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"orgName": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"projectId": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"requestId": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"requestType": {
							"type": "long"
						},
						"signContractCode": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						},
						"signSourceType": {
							"type": "long"
						},
						"source": {
							"type": "long"
						},
						"tenantId": {
							"type": "text",
							"fields": {
								"keyword": {
									"type": "keyword",
									"ignore_above": 256
								}
							}
						}
					}
				},
				"startTime": {
					"type": "long"
				},
				"title": {
					"type": "text",
					"fields": {
						"keyword": {
							"type": "keyword",
							"ignore_above": 256
						}
					}
				},
				"updateTime": {
					"type": "long"
				}
			}
		}
	}
}
```

在使用**_match_phrase_**对**title**字段进行检索时出现了非预期结果。ES中有多个doc的title字段值为”**合同流程222_分包项目**“，但是使用“**合同流程222**”却无法命中。

根据官方文档的说法：

Elasticsearch performs text analysis when indexing or searching **text** fields.

If your index doesn’t contain text fields, no further setup is needed; you can skip the pages in this section.

也就是说需要字段类型为text的才可以使用分词检索，而我的title字段确实是text类型，没问题。那么问题就出现在分词器配置上。

查询DSL：

```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "match_phrase": {
                        "title": "合同流程222"
                    }
                }
            ]
        }
    }
}
```

查询结果为空：

```json
{
	"took": 4,
	"timed_out": false,
	"_shards": {
		"total": 4,
		"successful": 4,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 0,
			"relation": "eq"
		},
		"max_score": null,
		"hits": []
	}
}
```

下面是经过多次测试的汇总：

| 关键词       | 是否命中 |
| ------------ | -------- |
| 合同流程     | 是       |
| 合同         | 是       |
| 合同流程_    | 是       |
| 合同流程222_ | 是       |
| 222          | 否       |
| 合同流程222  | 否       |

分词测试：

POST /wf_process_instance/_analyze

```json
// 使用默认的standard分词进行分词
{
  "analyzer" : "standard",
  "text" : "合同流程222_分包项目"
}

// 分词结果
{
    "tokens": [
        {
            "token": "合",
            "start_offset": 0,
            "end_offset": 1,
            "type": "<IDEOGRAPHIC>",
            "position": 0
        },
        {
            "token": "同",
            "start_offset": 1,
            "end_offset": 2,
            "type": "<IDEOGRAPHIC>",
            "position": 1
        },
        {
            "token": "流",
            "start_offset": 2,
            "end_offset": 3,
            "type": "<IDEOGRAPHIC>",
            "position": 2
        },
        {
            "token": "程",
            "start_offset": 3,
            "end_offset": 4,
            "type": "<IDEOGRAPHIC>",
            "position": 3
        },
        {
            "token": "222_", // 数字被处理为一个词
            "start_offset": 4,
            "end_offset": 8,
            "type": "<NUM>",
            "position": 4
        },
        {
            "token": "分",
            "start_offset": 8,
            "end_offset": 9,
            "type": "<IDEOGRAPHIC>",
            "position": 5
        },
        {
            "token": "包",
            "start_offset": 9,
            "end_offset": 10,
            "type": "<IDEOGRAPHIC>",
            "position": 6
        },
        {
            "token": "项",
            "start_offset": 10,
            "end_offset": 11,
            "type": "<IDEOGRAPHIC>",
            "position": 7
        },
        {
            "token": "目",
            "start_offset": 11,
            "end_offset": 12,
            "type": "<IDEOGRAPHIC>",
            "position": 8
        }
    ]
}


```

通过分词测试可以看出**合同流程222_分包项目**中文字符分成了单字短语，但是数字部分却没有，数字部分处理为了一个短语**222_**。

真相大白：

match_phrase是短语匹配，也就是将输入的关键词经过分词器分词后与索引中的字段分词进行等值匹配，match_phrase_prefix则是前缀短语匹配，将输入的关键词经过分词器分词后与索引中的字段分词进行前缀匹配。

**”合同流程222_分包项目“**的数字部分在索引存储时被分词成了**”222_“**，我们使用”222“进行_match_phrase_（短语匹配）自然无法匹配索引中的**“222_”**，导致未命中数据。如果使用_match_phrase_prefix_是可以命中数据的，因为使用是短语前缀匹配，也就是“222”是“222_”的前缀。

