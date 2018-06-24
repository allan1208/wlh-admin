# wlh-admin
wlh admin workspace.

## 基础框架
[iView](https://github.com/iview/iview) v2.8.0

[iView Admin](https://github.com/iview/iview-admin) v1.3.1

## 主要依赖
vue v2.5.13

vue-router v3.0.1

vuex v3.0.1


## 安装运行
```bush
// git clone
git clone ...

// into directory
cd wlh-amind

// install dependencies
// 注意，此处如果使用淘宝镜像地址会报错，报错后使用npm官方库重新执行即可
npm install
// npm install --registry=https://registry.npm.taobao.org

// run local
npm run dev
```

## 打包部署
```bush
// build prod bundle
npm run build
```
完成上述命令后，本地项目文件夹中dist目录下的index.html文件和dist文件夹放到服务器nginx对应目录下即可。


