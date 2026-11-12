# Dify

## 总结

### 1. 查询启用的知识库文件及下载路径

```postgresql
SELECT
    d.name AS dataset_name,
    doc.name AS doc_name,
    ('http://172.17.1.110/chatbi/' || f.key) as url
FROM datasets d
JOIN documents doc ON d.id = doc.dataset_id
JOIN upload_files f ON f.id = (doc.data_source_info::json ->> 'upload_file_id')::UUID
WHERE doc.enabled = TRUE
  AND d.name IN ('钢银系统Q&A', 'AI客服题库', '钢银电商帮助中心', '公司内部规章制度', '培训文件-运营&客服')
order by  d.name
```

