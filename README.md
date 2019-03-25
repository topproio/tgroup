# T组博客

## 开发指南

全局安装 `Hexo`

```bash
$ sudo npm install -g hexo-cli
```

进入项目根目录，安装依赖

```bash
$ npm install
```

本地服务器

```bash
$ hexo server
```

部署到指定分支

```bash
# 发布到仓库指定的分支
# 写作分支：master，博客分支：gh-pages，配置可见 _config.yml 的 deploy 部分
# 请勿直接修改 gh-pages 分支
$ hexo clean && hexo deploy --generate
```

## 写作指南

新建文章

```bash
# 如果不指定 layout 默认 source/_posts
hexo new [layout] (文章名称)
```

## 工作目录

```bash
/
│   public         存放发布资源，不可修改
│   scaffolds      模版文件
│   source         存放写作资源
│   themes         主题文件
|   _config.yml    配置文件 https://hexo.io/zh-cn/docs/configuration
```

## 相关链接

- [Hexo](https://hexo.io/zh-cn/)
- [Tgroup](https://topproio.github.io/tgroup/)
