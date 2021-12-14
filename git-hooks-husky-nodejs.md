# 使用Husky & Nodejs自定义你的Git钩子

## 背景就是所谓产生需求的场景
有时我们需要使用hooks对git的各类操作进行控制，就有了所谓的钩子（hooks）。然后有个问题是hooks只会存在于本地不会推送到远程仓库，导致无法团队间共享，维护是个大问题。因此 **Husky** 应运而生。

## 准备工作

初始化git
```bash
git init      
```

初始化package.json
```bash     
npm init --y   
```
安装husk@3
```bash
npm install --save-dev husky@3
```

## 修改package.json
```javascript
{
  "name": "git-hooks-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
     "husky": "^3.1.0",
  },
  "husky": {
    "hooks": {
      "pre-commit": "node pre-commit"
    }
  }
}

```

## 编写项目根目录下的pre-commit.js
```javascript
const execSync = require('child_process').execSync;
try {
    console.log('project.config.json >>>>>>>> 检查开始');
    // 检查暂存区是否存在修改appid
    const result = execSync(`git diff --staged project.config.json`, { encoding: 'utf-8' });
    const appid = result.match(/appid/g);
    if (appid && appid.length > 1) {
        console.log('appid 非法改动，无法提交')
        process.exit(1);
    }
} catch (e) {
    console.log('project.config.json 读取异常', e)
    process.exit(0)
}
console.log('project.config.json >>>>>>>> 检查结束')
process.exit(0)
```

## 结果
```bash
➜  motao-mini git:(feature/#引入hooks检查) ✗ git commit -m test_message
husky > pre-commit (node v14.15.1)
project.config.json >>>>>>>> 检查开始
appid 非法改动，无法提交
husky > pre-commit hook failed (add --no-verify to bypass)
```

## 各类奇怪问题汇总
- husky hooks not work（钩子不生效的），执行以下命令
```bash
rm -rf .git/hooks
npm install -D husky
```
- 命令行生效，但GUI(Tower,SourceTree,Sublime)之类的不生效的
```bash
npm uninstall husky && npm install --save-dev husky@3
```