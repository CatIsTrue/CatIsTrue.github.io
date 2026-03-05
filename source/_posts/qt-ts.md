# Qt多语言处理

## 环境
利用Qt原生模块

## tr函数引入
```cpp
QString text=tr("Measure");
```

## 语言模块
通过Qt Command Prompt

（1）命令行执行生成ts文件
```javascript
lupdate yourProject.pro -ts translations/zh_CN.ts
```
（2）手动修改ts文件的中文翻译

（3）生成qm文件
```javascript
lrelease translations\zh_CN.ts -qm translations\zh_CN.qm
```