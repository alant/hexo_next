# This is the editing branch of ahtang.com

Usage:
```bash
# write article locally then run the following to check locally
hexo server --config source/_data/next.yml 
# generate the necessary files to deploy to github
hexo generate --config source/_data/next.yml
# Deploy to github personal page:
hexo deploy --config source/_data/next.yml
```
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
```
