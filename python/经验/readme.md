# json.dumps错误

### 'utf8' codec can't decode byte解决方案

```python
value = unicode( value, errors='ignore')
```

