## 自动发布 Hexo 博客到 GitHub Pages

使用 Hexo 并通过 GitHub CI（GitHub Actions）实现自动发布，可按照以下步骤操作，实现推送代码后自动构建并部署到 GitHub Pages：

**1. 准备工作**

① 本地环境搭建
- 安装 [Node.js](https://nodejs.org/) 和 [Git](https://git-scm.com/)
- 全局安装 Hexo, 以及渲染器插件：
  ```bash
  npm install -g hexo-cli
  npm install hexo-renderer-pug --save
  ```

② 创建 Hexo 博客
```bash
# 初始化博客
hexo init my-blog
cd my-blog

# 安装依赖
npm install

# 本地预览（访问 http://localhost:4000）
hexo s
```

③ 关联 GitHub 仓库
1. 在 GitHub 新建仓库（仓库名为 `<username>.github.io`表示根仓库，便于直接部署 GitHub Pages）
2. 将本地博客关联到远程仓库：
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/<username>/<username>.github.io.git
   git push -u origin main
   ```
3. 需要在仓库的Settings中开启Pages服务，将分支设置为`gh-pages`，并将路径设置为`/(root)`。


**2. 配置 GitHub Actions 自动部署**
GitHub Actions 可通过工作流文件实现代码推送后自动构建并部署 Hexo 博客。

① 创建工作流文件
在本地博客目录中创建以下文件结构：
```
my-blog/
└── .github/
    └── workflows/
        └── deploy.yml
```

② 编写工作流配置（`deploy.yml`）
```yaml
name: Deploy Hexo to GitHub Pages

# 触发条件：推送到 master 分支时执行
on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest  # 使用 Ubuntu 环境

    steps:
      # # 1. 拉取代码   # 取消使用子模块，可能对主题进行编辑
      # - name: Checkout code
      #   uses: actions/checkout@v4
      #   with:
      #     submodules: true  # 若使用主题子模块，需开启

      # 2. 安装 Node.js
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20.x  # 指定 Node 版本

      # 3. 安装 Hexo 依赖
      - name: Install dependencies
        run: |
          npm install -g hexo-cli
          npm install

      # 4. 构建静态文件
      - name: Generate static files
        run: hexo clean && hexo generate  # 清理缓存并生成

      # 5. 部署到 GitHub Pages
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4  # 第三方部署工具
        with:
          # 部署到 gh-pages 分支
          publish_branch: gh-pages
          # 生成的静态文件目录
          publish_dir: ./public
          #  GitHub 访问令牌（需提前配置）
          github_token: ${{ secrets.PERSONAL_TOKEN }}
          # 强制推送（可选）
          force_orphan: true
```


**3. 配置 GitHub 访问权限**
1. 生成 GitHub 个人访问令牌（PAT）：
   - 进入 GitHub 个人设置 → **Settings → Developer settings → Personal access tokens → Tokens (classic)**
   - 点击 **Generate new token(classic)**，勾选 `repo、workflow` 权限（用于推送代码），生成令牌后复制保存。

2. 在仓库中配置令牌：
   - 进入博客仓库 → **Settings → Secrets and variables → Actions → New repository secret**
   - 名称填写 `PERSONAL_TOKEN`，值粘贴刚才生成的令牌。


**4. 测试自动部署**
1. 推送本地代码到 GitHub main 分支：
   ```bash
   git add .github/workflows/deploy.yml  # 添加工作流文件
   git commit -m "Add GitHub Actions workflow"
   git push
   ```

2. 查看部署状态：
   - 进入仓库 → **Actions → Actions**，查看工作流是否运行成功。
   - 成功后，访问 `https://<username>.github.io` 即可看到部署的博客。


我建议参照教程完整过一遍。  https://docs.anheyu.com/intro.html

## 使用安知鱼主题（anzhiyu）

1. **查看主题官方文档**
   主题的详细配置说明请参考：[Hexo Theme Anzhiyu 官方文档](https://github.com/anzhiyu-c/hexo-theme-anzhiyu?tab=readme-ov-file)

2. **添加主题到项目中**
   ```bash
   git clone -b main https://github.com/anzhiyu-c/hexo-theme-anzhiyu.git themes/anzhiyu
   ```

todo: https://docs.anheyu.com/global/extra.html