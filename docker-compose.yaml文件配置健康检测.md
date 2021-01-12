docker-compose.yaml文件配置健康检测：

```bash
  test: ["CMD-SHELL", "curl -fs http://localhost:8016/v2/file-service-rest/swagger-ui.html >> /dev/null"]
```

或者

   ```bash
   test: ["CMD-SHELL", "curl -fs http://localhost:8016/v2/file-service-rest/swagger-ui.html >> /dev/null || exit 1"]
   ```

