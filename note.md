```shell
// 以项目名electron-vue3-ts-vite-test2为例
npm create vite@latest electron-vue3-ts-vite-test2

// 终端
npm create vite@latest electron-vue3-ts-vite-test2

> npx
> create-vite electron-vue3-ts-vite-test2

√ Select a framework: » Others
√ Select a variant: » create-electron-vite ↗

> npx
> create-electron-vite electron-vue3-ts-vite-test2

√ Project template: » Vue

Scaffolding project in E:\coding\elelctronProjects\musicVue\electron-vue3-ts-vite-test2...

Done. Now run:

  cd electron-vue3-ts-vite-test2
  npm install
  npm run dev

```

```shell
npm install //完成初始化
npm run dev //试运行确认是否有问题
```

```shell
//安装electron
npm install electron -D
//打包工具
npm install electron-builder -D
//开发工具
npm install electron-devtools-installer -D
//集成Vite和Electron，方便后续在渲染进程中使用Node API或Electron API
npm install vite-plugin-electron -D
npm install vite-plugin-electron-renderer -D
 
//快速删除某些文件和文件夹
npm install rimraf -D    
```

