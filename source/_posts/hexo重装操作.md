# hexo重装操作
1. 安装node.js
2. 切换node.js 源
``` node.js
npm install nrm -g
nrm use taobao
nrm test
```
3. 生成公钥并且在github中添加公钥
``` shell
ssh-keygen -t rsa -C "2878466@qq.com"
```

4. 通过git克隆hexo
``` shell
git clone git@github.com:dgqg1223/dgqg1223.github.io.git
```

5. 初始化hexo并
``` shell
npm install -g hexo-cli
hexo init
npm install    
```

6. 设置git信息
``` shell
git config --global user.name "CHEN"    
git config --global user.email "2878466@qq.com"    
```
