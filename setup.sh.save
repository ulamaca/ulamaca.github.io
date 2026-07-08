#!/bin/bash
# as-folio 極簡化腳本:在 GitHub Codespace 終端機執行
# 用法: bash setup.sh
set -u

echo "=== 1/5 修改站台設定 ==="
python3 - <<'EOF'
import re
p = 'src/config/site.ts'
s = open(p).read()

# 導覽列只留 about / blog / publications
s = re.sub(
    r"\{ label: 'about', href: '/' \},.*?\] as NavItem\[\]",
    "{ label: 'about', href: '/' },\n      { label: 'blog', href: '/blog/' },\n      { label: 'publications', href: '/publications/' },\n    ] as NavItem[]",
    s, flags=re.S)

# 首頁移除公告 / 最新文章 / 精選論文區塊
s = re.sub(r"(announcements: \{[^}]*?enabled: )true", r"\1false", s)
s = re.sub(r"(latestPosts: \{[^}]*?enabled: )true", r"\1false", s)
s = re.sub(r"(selectedPapers: \{[^}]*?enabled: )true", r"\1false", s)

# 關閉非必要功能(要恢復就把 false 改回 true)
for k in ['progressBar', 'mediumZoom', 'masonry', 'socialShare', 'search']:
    s = re.sub(r"(\b%s: )true" % k, r"\1false", s)

# 清空 blog 列表頁的示範 tag 徽章
s = s.replace(
    "displayTags: ['formatting', 'images', 'links', 'math', 'code', 'blockquotes']",
    "displayTags: [] as string[]")

open(p, 'w').write(s)
print('site.ts done')
EOF

echo "=== 2/5 刪除不需要的頁面 ==="
rm -rf src/pages/books.astro src/pages/cv.astro src/pages/news.astro \
       src/pages/projects src/pages/repositories.astro \
       src/pages/teaching src/pages/people

echo "=== 3/5 精簡示範文章(只留數學/程式碼/格式三篇)==="
cd src/content/posts
for f in *; do
  case "$f" in
    math-typesetting.mdx|code-blocks.mdx|formatting-examples.mdx) ;;
    *) rm -f "$f" ;;
  esac
done
cd ../../..

echo "=== 4/5 更新 about 佔位內容、清理素材與多餘 workflow ==="
cat > src/data/about.mdx <<'ABOUT'
（這裡放你的自我介紹——建議 2-3 段:第一段一句話定位你是誰、研究什麼;第二段研究興趣與目前工作;第三段學歷背景。）

範例:我是 ○○ 大學○○系的博士生,研究方向為 ○○○。我關注的核心問題是 ○○○。

聯絡方式與學術檔案連結會自動顯示在下方(於 `src/config/site.ts` 的 socials 區塊設定)。
ABOUT

rm -rf public/assets/audio public/assets/video \
       public/assets/img/book_covers \
       public/assets/img/planck.jpg public/assets/img/curie.jpg \
       public/assets/img/heisenberg.jpg public/assets/img/bohr.jpg \
       public/assets/img/as_folio_homepage.png public/assets/img/as_folio_blog.png \
       public/assets/img/as_folio_search.png
rm -rf e2e playwright.config.ts release-please-config.json \
       .github/workflows/release-please.yml .github/workflows/ci.yml

# 自動偵測 GitHub 帳號,repo 為「帳號.github.io」根網域部署,base 設為空
OWNER=$(git remote get-url origin | sed -E 's#.*[:/]([^/]+)/([^/.]+)(\.git)?$#\1#')
sed -i "s#vars.ASTRO_SITE || '[^']*'#vars.ASTRO_SITE || 'https://${OWNER}.github.io'#" .github/workflows/deploy.yml
sed -i "s#vars.ASTRO_BASE || '[^']*'#vars.ASTRO_BASE || ''#" .github/workflows/deploy.yml
echo "部署網址將是: https://${OWNER}.github.io"

echo "=== 5/5 提交並推送 ==="
git add -A
git commit -m "Minimize to about/blog/publications"
git push

echo ""
echo "完成!最後一步:到 repo 的 Settings → Pages → Source 選 'GitHub Actions'"
echo "然後到 Actions 分頁看部署進度,綠色勾勾出現後網站就上線了。"
