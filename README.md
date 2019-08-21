#### 环境安装：

- 安装git，设置ssh key
- 安装node.js
- 安装hexo

#### 操作步骤

- 任意文件夹下克隆代码`git clone https://github.com/lucas234/lucas234.github.io`
- 进入到克隆文件夹
```
cd lucas234.github.io.git
npm install
npm install hexo-deployer-git --save
```
- 生成、部署

```
hexo clean
hexo g
hexo s
# 如果执行后无法启动本地服务，在当前目录中，执行以下语句
# 例如: 访问 http://localhost:4000/,返回 Cannot GET /
npm install
npm install hexo --save
npm install hexo-server --save
```

#### Tips：hexo 命令

```
hexo new "postName"  # 新建文章
hexo new page "pageName"  # 新建页面
hexo generate  # 生成静态页面至public目录
hexo server  # 开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy  # 将.deploy目录部署到GitHub
hexo help   # 查看帮助
hexo version  # 查看Hexo的版本
hexo deploy -g  # 生成加部署
hexo server -g  # 生成加预览
# 命令的简写
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

