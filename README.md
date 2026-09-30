# [Maples' Blog](https://blog.maples31.com/)

## Command

```bash
# New a post/draft/page
hexo new post <title>
hexo new page --path about/me "About me"
hexo new draft <title>

# Generate
hexo generate

# Server
hexo server

# Deploy
hexo deploy

# Clean
hexo clean

# Publish
hexo publish
```

## Notes

## 图片与私有 OSS 回源

- 图片及个人介绍页原有的 OSS CSS、字体和图标脚本保存在 `source/assets/imported/`，构建后上传到 `blog-maples31/assets/imported/`。页面使用 `/assets/imported/...` 根路径引用，经 `blog.maples31.com` 访问。
- 新图片放入 `source/images/`，文章使用 `![说明](/images/文件名.png)`；不要写 OSS 公网地址。
- GitHub Actions 会将 `public/` 中的图片和页面一起上传到当前网站 Bucket。图片导入后必须先完成部署，再关闭旧资源的公共访问。
- CDN 控制台：域名管理 → `blog.maples31.com` → 回源配置 → 阿里云 OSS 私有 Bucket 回源 → 授权并开启同账号回源。源站与回源 HOST 应为 `blog-maples31.oss-cn-hongkong.aliyuncs.com`。
- CDN 私有回源下，不应依赖 OSS 静态网站托管自动补默认首页。目录 URL（包括 `/` 和文章目录）需要在 CDN 回源时补上 `index.html`，例如 `/2024/05/11/Arch-显卡配置/` 回源 `/2024/05/11/Arch-显卡配置/index.html`。如果当前已用边缘函数或重写实现，沿用现有配置。
- 验证网站首页、文章目录和图片均可访问后，将 Bucket ACL 改为私有；图片对象 ACL 也应为私有或继承 Bucket，删除允许匿名读取的 Bucket Policy。最终确认未签名的 OSS 图片直链返回 403，而 CDN 图片返回 200。
- 私有回源限制的是 OSS 公网直链读取，CDN 域名上的图片仍然公开可访问。
- 部署若出现 `InvalidAccessKeyId`，在仓库 Actions Secrets 更新 `OSS_KEY_ID`、`OSS_KEY_SECRET`，然后重新运行部署。

参考：[阿里云 OSS 私有 Bucket 回源](https://help.aliyun.com/zh/cdn/user-guide/grant-alibaba-cloud-cdn-access-permissions-on-private-oss-buckets)。

- [hexo](https://hexo.io/docs/)
- [hexo-github-card](https://github.com/Gisonrg/hexo-github-card)


