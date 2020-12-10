## 教你如何3分钟把VUE项目打包成apk

**必要条件**

使用vue-cli3搭建的项目，

**工具**

HBuilder X，他的图标长这个样子，[点击下载](https://www.dcloud.io/hbuilderx.html)

![](https://www.dengzhanyong.com/PHP/images/1604320508.png)

做前端的大多数小伙伴们都应该知道，使用起来轻巧、急速，但是他主要是针对于VUE生态打造的，相对于 vscode 缺少了丰富的插件支持以及多语言编译的支持。但是它也有vscode无法满足的功能，比如说今天我们就要用它来把vue项目打包为 apk。

#### 打包步骤

1. 执行 `npm run build` 打包vue项目
2. 下载并安装HBuilder X
3. 依次点击文件》新建》项目，选择 `5+APP(A)` 选项，并填写好项目名称，选择项目保存位置，选择默认模板

![](https://www.dengzhanyong.com/PHP/images/1604321138.png)

![](https://www.dengzhanyong.com/PHP/images/1604321513.png)

4. 将 vue 打包后的dist 目录下的所有内容拷贝到刚才创建的项目目录下

5. 点击 `manifest.json` 文件

   - 选择基础配置，填写AppID(需要自己申请)和应用名称

   ![](https://www.dengzhanyong.com/PHP/images/1604321654.png)

   - 图标设置

     你可以“点击自动生成所有图标并替换”，系统会自动为你生成各种尺寸的图标

     ![](https://www.dengzhanyong.com/PHP/images/1604321842.png)

   - 其他的配置可以就是用默认的配置，不用去管它

6. 点击发行》原生APP云打包，勾选Android，选择“使用公测测试证书”

   ![](https://www.dengzhanyong.com/PHP/images/1604322025.png)



完成这一步后只需要耐心的等待，打包完成后会自动返回下载链接，你只需要下载下来然后和正常的APP一样安装即可。

![](https://www.dengzhanyong.com/PHP/images/1604389530.png)



打开应用，可以看到里面的内容和我们在浏览器中看到的效果是一样的

![](https://www.dengzhanyong.com/PHP/images/1604322334.jpg)

![](https://www.dengzhanyong.com/PHP/images/1604322322.jpg)



#### 注意点

1.  `vue.config.js` 中一定要配置

   ```javascript
   {
       publicPath: './',
   }
   ```

2. 路由模式使用 `history` 模式

3. 打开新页面不能使用 `window.open` 方法，因为在这里面没有 `window` 对象，不然你看到的将会是这个样子

   ![](https://www.dengzhanyong.com/PHP/images/1604389757.png)



