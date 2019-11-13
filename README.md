## 个人博客

### 启动

```
npm run dev
```
hexo 需要安装第三方插件（`hexo-browsersync`）来`autoload`
```
npm run dev:watch
```

### 新闻
```
hexo new xxx
```

### 生成文件
```
hexo generate --watch
```
### 生成文件后部署
```
hexo generate --deploy || hexo deploy --generate
hexo g -d || hexo d -g 简写
```
### 部署
```
npm run deploy
```