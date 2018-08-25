# alant.github.io
This is the editing branch.

write article locally then run: hexo server --config source/_data/next.yml to check
run: hexo generate --config source/_data/next.yml to generate the necessary files to deploy to github
run: hexo deploy --config source/_data/next.yml to deploy automatically to github personal page

For new clones:
```bash
# install all dependencies:
npm i 
# install the theme called next
git clone https://github.com/theme-next/hexo-theme-next themes/next
# install dependency fancybox 
cd themes/next
git clone https://github.com/theme-next/theme-next-fancybox3 source/lib/fancybox
# now you can run local server:
hexo server --config source/_data/next.yml
