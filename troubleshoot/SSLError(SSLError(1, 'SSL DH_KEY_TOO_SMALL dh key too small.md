
해결코드
```python
requests.packages.urllib3.util.ssl_.DEFAULT_CIPHERS = 'ALL:@SECLEVEL=1'
```

원인이 되었던 변경
https://github.com/python/cpython/pull/25778/files#diff-89879be484d86da4e77c90d5408b2e10190ee27cc86337cd0f8efc3520d60621R172



https://velog.io/@jungbumwoo/%EB%94%94%ED%94%BC-%ED%97%AC%EB%A8%BC-DH-key-Diffie-Hellman-protocol-%EC%9D%B4%EB%9E%80