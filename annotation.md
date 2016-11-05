# Annotations

Used to attach arbitrary non-identifying metadata, for retrieval by API clients. May be large, structured or unstructured, may include characters not permitted by labels. Like labels, annotations are key-value maps

```
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```

Possible information that could be recorded in annotations:
- build/release/image information (timestamps, release ids, git branch, PR numbers, image hashes, registry address, etc.)
- pointers to logging/monitoring/analytics/audit repos
- client library/tool information (e.g. for debugging purposes – name, version, build info)
- phone/pager number(s) of person(s) responsible, or directory entry where that info could be found, such as a team website
