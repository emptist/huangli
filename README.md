# 时轮运转

> 藏历时轮日历 + 每日能量 + 人生进度条

## 在线访问

https://huangli.netlify.app

## 功能

- 藏历/佛历日期显示
- 干支纪年（丙午年壬辰月戊寅日）
- 人生进度条（已活天数 / 职场天数 / 距退休）
- 今日指南（推荐/避开）
- 四大能量场：今日桃花、颜值能量、财富能量、身心状态
- 幸运色、幸运数、旺财方、今日饮品
- 每日一言
- 黄财神心咒音乐播放
- 32朵飘动莲花背景

## 技术栈

- 纯静态 HTML + CSS + JavaScript
- [lunar-javascript](https://github.com/6tail/lunar-javascript) 藏历/农历库
- Canvas 莲花动画背景
- localStorage 本地存储生日

## 本地运行

```bash
cd /Users/jk/gits/hub/tools_ai/huangli
python3 -m http.server 8765
# 访问 http://localhost:8765
```

## 部署方式

### Netlify（推荐）

```bash
# 安装
pnpm add -g netlify-cli

# 部署
cd /Users/jk/gits/hub/tools_ai/huangli
netlify deploy --prod
```

### Surge

```bash
pnpm add -g surge
surge .
```

### Vercel

```bash
pnpm add -g vercel
vercel
```

### GitHub Pages

1. 打开 https://github.com/emptist/huangli/settings/pages
2. Source 选 Deploy from a branch
3. Branch 选 master / root
4. Save
5. 访问 https://emptist.github.io/huangli/

## 更新代码后重新部署

```bash
cd /Users/jk/gits/hub/tools_ai/huangli
git add . && git commit -m "update" && git push
netlify deploy --prod
```

## GitHub 仓库

https://github.com/emptist/huangli

## 音乐来源

《随愿聚福》- 黄财神心咒 × 现代流行旋律
https://www.bilibili.com/video/BV1yKR6YYENx/

心咒：嗡 藏巴拉 扎连札 耶 梭哈
