docker-compose显示中文

将中文字符文件放在fonts目录下，在docker-compose文件中添加以下： 

```bash
volumes:

- ./fonts:/usr/share/fonts
```





再执行

```bash
# docker-compose up -d --build
```

