[Puppeteer](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fpuppeteer%2Fpuppeteer&source=article&objectId=1942245) 是 Chrome 开发团队在 2017 年发布的一个 Node.js 包，同时还有 Headless Chrome。用来模拟 Chrome 浏览器的运行。它提供了高级API来通过 DevTools 协议控制无头 Chrome 或 Chromium ，它也可以配置为使用完整（非无头）Chrome 或 Chromium。

![](https://ask.qcloudimg.com/http-save/yehe-6710420/13c5c5b523ebd4f1b1ccd32c1418a3da.png)

学习 Puppeteer 之前我们先来了解一下 [Chrome DevTool Protocol](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fchromedevtools.github.io%2Fdevtools-protocol%2F&source=article&objectId=1942245) 和 [Headless Chrome](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27100187&source=article&objectId=1942245)。

#### Chrome DevTool Protocol 是什么

+   CDP 基于 WebSocket，利用 WebSocket 实现与浏览器内核的快速数据通道。
+   CDP 分为多个域（DOM，Debugger，Network，Profiler，Console...），每个域中都定义了相关的命令和事件（Commands and Events）。
+   我们可以基于 CDP 封装一些工具对 Chrome 浏览器进行调试及分析，比如我们常用的 “Chrome 开发者工具” 就是基于 CDP 实现的。
+   很多有用的工具都是基于 CDP 实现的，比如 [Chrome 开发者工具](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fdevelopers.google.cn%2Fweb%2Ftools%2Fchrome-devtools%2F&source=article&objectId=1942245)，[chrome-remote-interface](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fcyrus-and%2Fchrome-remote-interface&source=article&objectId=1942245)，[Puppeteer](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fpuppeteer%2Fpuppeteer&source=article&objectId=1942245) 等。

#### Headless Chrome 是什么

+   可以在无界面的环境中运行 Chrome。
+   通过命令行或者程序语言操作 Chrome。
+   无需人的干预，运行更稳定。
+   在启动 Chrome 时添加参数 --headless，便可以 headless 模式启动 Chrome。
+   chrome 启动时可以加一些什么参数，大家可以点击[这里](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fpeter.sh%2Fexperiments%2Fchromium-command-line-switches%2F&source=article&objectId=1942245)查看。

总而言之 Headless Chrome 就是 Chrome 浏览器的无界面形态，可以在不打开浏览器的前提下，使用所有 Chrome 支持的特性运行你的程序。

#### Puppeteer 是什么

+   Puppeteer 是 Node.js 工具引擎。
+   Puppeteer 提供了一系列 API，通过 Chrome DevTools Protocol 协议控制 Chromium/Chrome 浏览器的行为。
+   Puppeteer 默认情况下是以 headless 启动 Chrome 的，也可以通过参数控制启动有界面的 Chrome。
+   Puppeteer 默认绑定最新的 Chromium 版本，也可以自己设置不同版本的绑定。
+   Puppeteer 让我们不需要了解太多的底层 CDP 协议实现与浏览器的通信。

#### Puppeteer 能做什么

官方介绍：您可以在浏览器中手动执行的大多数操作都可以使用 Puppeteer 完成！示例：

+   生成页面的屏幕截图和PDF。
+   爬取 SPA 或 SSR 网站。
+   自动化表单提交，UI测试，键盘输入等。
+   创建最新的自动化测试环境。使用最新的JavaScript和浏览器功能，直接在最新版本的Chrome中运行测试。
+   捕获站点的时间线跟踪，以帮助诊断性能问题。
+   测试Chrome扩展程序。
+   ...

#### Puppeteer API 分层结构

Puppeteer 中的 API 分层结构基本和浏览器保持一致，下面对常使用到的几个类介绍一下：

![](https://ask.qcloudimg.com/http-save/yehe-6710420/bb3fa5661e29ac8a8dbcc23bfe4f5a0f.png)

+   **Browser**： 对应一个浏览器实例，一个 Browser 可以包含多个 BrowserContext
+   **BrowserContext**： 对应浏览器一个上下文会话，就像我们打开一个普通的 Chrome 之后又打开一个隐身模式的浏览器一样，BrowserContext 具有独立的 Session(cookie 和 cache 独立不共享)，一个 BrowserContext 可以包含多个 Page
+   **Page**：表示一个 Tab 页面，通过 browserContext.newPage()/browser.newPage() 创建，browser.newPage() 创建页面时会使用默认的 BrowserContext，一个 Page 可以包含多个 Frame
+   **Frame**: 一个框架，每个页面有一个主框架（page.MainFrame()）,也可以多个子框架，主要由 iframe 标签创建产生的
+   **ExecutionContext**： 是 javascript 的执行环境，每一个 Frame 都一个默认的 javascript 执行环境
+   **ElementHandle**: 对应 DOM 的一个元素节点，通过该该实例可以实现对元素的点击，填写表单等行为，我们可以通过选择器，xPath 等来获取对应的元素
+   **JsHandle**：对应 DOM 中的 javascript 对象，ElementHandle 继承于 JsHandle，由于我们无法直接操作 DOM 中对象，所以封装成 JsHandle 来实现相关功能
+   **CDPSession**：可以直接与原生的 CDP 进行通信，通过 session.send 函数直接发消息，通过 session.on 接收消息，可以实现 Puppeteer API 中没有涉及的功能
+   **Coverage**：获取 JavaScript 和 CSS 代码覆盖率
+   **Tracing**：抓取性能数据进行分析
+   **Response**： 页面收到的响应
+   **Request**： 页面发出的请求

#### Puppeteer 安装与环境

> 注意：在v1.18.1之前，Puppeteer至少需要Node v6.4.0。从v1.18.1到v2.1.0的版本依赖于Node 8.9.0+。从v3.0.0开始，Puppeteer开始依赖于Node 10.18.1+。若要使用 [async / await](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fasync&source=article&objectId=1942245)，只有Node v7.6.0或更高版本才支持。

Puppeteer是一个node.js包，所以安装很简单：

```javascript
npm install puppeteer
// 或者
yarn add puppeteer
```

> npm 在安装 puppeteer 的时候可能会报错！这是由于外网导致，使用访问国外网站或者使用淘宝镜像 cnpm 安装可解决。

安装Puppeteer时，它将下载 Chromium 的最新版本。从1.7.0版开始，官方发布了该 [puppeteer-core](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fpuppeteer-core&source=article&objectId=1942245) 软件包，默认情况下不会下载任何浏览器，用于启动现有的浏览器或连接到远程浏览器。需要注意安装的 puppeteer-core 版本与打算连接的浏览器兼容。

#### Puppeteer 使用

##### Case1: 截图

我们使用 Puppeteer 既可以对某个页面进行截图，也可以对页面中的某个元素进行截图：

```javascript
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  //设置可视区域大小,默认的页面大小为800x600分辨率
  await page.setViewport({width: 1920, height: 800});
  await page.goto('https://www.baidu.com/');
  //对整个页面截图
  await page.screenshot({
      path: './files/baidu_home.png',  //图片保存路径
      type: 'png',
      fullPage: true //边滚动边截图
      // clip: {x: 0, y: 0, width: 1920, height: 800}
  });
  //对页面某个元素截图
  let element = await page.$('#s_lg_img');
  await element.screenshot({
      path: './files/baidu_logo.png'
  });
  await page.close();
  await browser.close();
})();
```

我们怎么去获取页面中的某个元素呢？

+   `page.$('#uniqueId')`：获取某个选择器对应的第一个元素
+   `page.$$('div')`：获取某个选择器对应的所有元素
+   `page.$x('//img')`：获取某个 xPath 对应的所有元素
+   `page.waitForXPath('//img')`：等待某个 xPath 对应的元素出现
+   `page.waitForSelector('#uniqueId')`：等待某个选择器对应的元素出现

##### Case2: 模拟用户操作

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({
        slowMo: 100,    //放慢速度
        headless: false, //开启可视化
        defaultViewport: {width: 1440, height: 780},
        ignoreHTTPSErrors: false, //忽略 https 报错
        args: ['--start-fullscreen'] //全屏打开页面
    });
    const page = await browser.newPage();
    await page.goto('https://www.baidu.com/');
    //输入文本
    const inputElement = await page.$('#kw');
    await inputElement.type('hello word', {delay: 20});
    //点击搜索按钮
    let okButtonElement = await page.$('#su');
    //等待页面跳转完成，一般点击某个按钮需要跳转时，都需要等待 page.waitForNavigation() 执行完毕才表示跳转成功
    await Promise.all([
        okButtonElement.click(),
        page.waitForNavigation()  
    ]);
    await page.close();
    await browser.close();
})();
```

那么 ElementHandle 都提供了哪些操作元素的函数呢？

+   `elementHandle.click()`：点击某个元素
+   `elementHandle.tap()`：模拟手指触摸点击
+   `elementHandle.focus()`：聚焦到某个元素
+   `elementHandle.hover()`：鼠标 hover 到某个元素上
+   `elementHandle.type('hello')`：在输入框输入文本

##### Case3: 植入 javascript 代码

Puppeteer 最强大的功能是，你可以在浏览器里执行任何你想要运行的 javascript 代码。下面代码是对百度首页新闻推荐爬取数据的例子。

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.goto('https://www.baidu.com/');
    //通过 page.evaluate 在浏览器里执行代码
    const resultData = await page.evaluate(async () =>  {
      let data = {};
      const ListEle = [...document.querySelectorAll('#hotsearch-content-wrapper .hotsearch-item')];
      data = ListEle.map((ele) => {
        const urlEle = ele.querySelector('a.c-link');
        const titleEle = ele.querySelector('.title-content-title');
        return {
          href: urlEle.href,
          title: titleEle.innerText,
        };
      });
      return data;
    });
    console.log(resultData)
    await page.close();
    await browser.close();
})();
```

有哪些函数可以在浏览器环境中执行代码呢？

+   `page.evaluate(pageFunction[, ...args])`：在浏览器环境中执行函数
+   `page.evaluateHandle(pageFunction[, ...args])`：在浏览器环境中执行函数，返回 JsHandle 对象
+   `page.$$eval(selector, pageFunction[, ...args])`：把 selector 对应的所有元素传入到函数并在浏览器环境执行
+   `page.$eval(selector, pageFunction[, ...args])`：把 selector 对应的第一个元素传入到函数在浏览器环境执行
+   `page.evaluateOnNewDocument(pageFunction[, ...args])`：创建一个新的 Document 时在浏览器环境中执行，会在页面所有脚本执行之前执行
+   `page.exposeFunction(name, puppeteerFunction)`：在 window 对象上注册一个函数，这个函数在 Node 环境中执行，有机会在浏览器环境中调用 Node.js 相关函数库

##### Case4: 请求拦截

请求在有些场景下很有必要，拦截一下没必要的请求提高性能，我们可以在监听 Page 的 request 事件，并进行请求拦截，前提是要开启请求拦截 `page.setRequestInterception(true)`。

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    const blockTypes = new Set(['image', 'media', 'font']);
    await page.setRequestInterception(true); //开启请求拦截
    page.on('request', request => {
        const type = request.resourceType();
        const shouldBlock = blockTypes.has(type);
        if(shouldBlock){
            //直接阻止请求
            return request.abort();
        }else{
            //对请求重写
            return request.continue({
                //可以对 url，method，postData，headers 进行覆盖
                headers: Object.assign({}, request.headers(), {
                    'puppeteer-test': 'true'
                })
            });
        }
    });
    await page.goto('https://www.baidu.com/');
    await page.close();
    await browser.close();
})();
```

那 page 页面上都提供了哪些事件呢？

+   `page.on('close')` 页面关闭
+   `page.on('console')` console API 被调用
+   `page.on('error')` 页面出错
+   `page.on('load')` 页面加载完
+   `page.on('request')` 收到请求
+   `page.on('requestfailed')` 请求失败
+   `page.on('requestfinished')` 请求成功
+   `page.on('response')` 收到响应
+   `page.on('workercreated')` 创建 webWorker
+   `page.on('workerdestroyed')` 销毁 webWorker

##### Case5: 获取 WebSocket 响应

Puppeteer 目前没有提供原生的用于处理 WebSocket 的 API 接口，但是我们可以通过更底层的 Chrome DevTool Protocol (CDP) 协议获得

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    //创建 CDP 会话
    let cdpSession = await page.target().createCDPSession();
    //开启网络调试,监听 Chrome DevTools Protocol 中 Network 相关事件
    await cdpSession.send('Network.enable');
    //监听 webSocketFrameReceived 事件，获取对应的数据
    cdpSession.on('Network.webSocketFrameReceived', frame => {
        let payloadData = frame.response.payloadData;
        if(payloadData.includes('push:query')){
            //解析payloadData，拿到服务端推送的数据
            let res = JSON.parse(payloadData.match(/\{.*\}/)[0]);
            if(res.code !== 200){
                console.log(`调用websocket接口出错:code=${res.code},message=${res.message}`);
            }else{
                console.log('获取到websocket接口数据：', res.result);
            }
        }
    });
    await page.goto('https://netease.youdata.163.com/dash/142161/reportExport?pid=700209493');
    await page.waitForFunction('window.renderdone', {polling: 20});
    await page.close();
    await browser.close();
})();
```

##### Case6: 如何抓取 iframe 中的元素

一个 Frame 包含了一个执行上下文（Execution Context），我们不能跨 Frame 执行函数，一个页面中可以有多个 Frame，主要是通过 iframe 标签嵌入的生成的。其中在页面上的大部分函数其实是 page.mainFrame().xx 的一个简写，Frame 是树状结构，我们可以通过 frame.childFrames() 遍历到所有的 Frame，如果想在其它 Frame 中执行函数必须获取到对应的 Frame 才能进行相应的处理

以下是在登录 188 邮箱时，其登录窗口其实是嵌入的一个 iframe，以下代码时我们在获取 iframe 并进行登录

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({headless: false, slowMo: 50});
    const page = await browser.newPage();
    await page.goto('https://www.188.com');

    for (const frame of page.mainFrame().childFrames()){
        //根据 url 找到登录页面对应的 iframe
        if (frame.url().includes('passport.188.com')){
            await frame.type('.dlemail', 'admin@admin.com');
            await frame.type('.dlpwd', '123456');
            await Promise.all([
                frame.click('#dologin'),
                page.waitForNavigation()
            ]);
            break;
        }
    }
    await page.close();
    await browser.close();
})();
```

##### Case7: 页面性能分析

Puppeteer 提供了对页面性能分析的工具，目前功能还是比较弱的，只能获取到一个页面性能执行的数据，如何分析需要我们自己根据数据进行分析，据说在 2.0 版本会做大的改版： - 一个浏览器同一时间只能 trace 一次 - 在 devTools 的 Performance 可以上传对应的 json 文件并查看分析结果 - 我们可以写脚本来解析 trace.json 中的数据做自动化分析 - 通过 tracing 我们获取页面加载速度以及脚本的执行性能

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.tracing.start({path: './files/trace.json'});
    await page.goto('https://www.google.com');
    await page.tracing.stop();
    /*
        continue analysis from 'trace.json'
    */
    browser.close();
})();
```

##### Case8: 文件的上传和下载

在自动化测试中，经常会遇到对于文件的上传和下载的需求，那么在 Puppeteer 中如何实现呢？

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    //通过 CDP 会话设置下载路径
    const cdp = await page.target().createCDPSession();
    await cdp.send('Page.setDownloadBehavior', {
        behavior: 'allow', //允许所有下载请求
        downloadPath: 'path/to/download'  //设置下载路径
    });
    //点击按钮触发下载
    await (await page.waitForSelector('#someButton')).click();
    //等待文件出现，轮训判断文件是否出现
    await waitForFile('path/to/download/filename');

    //上传时对应的 inputElement 必须是<input>元素
    let inputElement = await page.waitForXPath('//input[@type="file"]');
    await inputElement.uploadFile('/path/to/file');
    browser.close();
})();
```

##### Case9: 跳转新 tab 页处理

在点击一个按钮跳转到新的 Tab 页时会新开一个页面，这个时候我们如何获取改页面对应的 Page 实例呢？可以通过监听 Browser 上的 targetcreated 事件来实现，表示有新的页面创建：

```javascript
let page = await browser.newPage();
await page.goto(url);
let btn = await page.waitForSelector('#btn');
//在点击按钮之前，事先定义一个 Promise，用于返回新 tab 的 Page 对象
const newPagePromise = new Promise(res => 
  browser.once('targetcreated', 
    target => res(target.page())
  )
);
await btn.click();
//点击按钮后，等待新tab对象
let newPage = await newPagePromise;
```

##### Case10: 模拟不同的设备

Puppeteer 提供了模拟不同设备的功能，其中 puppeteer.devices 对象上定义很多设备的配置信息，这些配置信息主要包含 viewport 和 userAgent，然后通过函数 page.emulate 实现不同设备的模拟

```javascript
const puppeteer = require('puppeteer');
const iPhone = puppeteer.devices['iPhone 6'];
puppeteer.launch().then(async browser => {
  const page = await browser.newPage();
  await page.emulate(iPhone);
  await page.goto('https://www.baidu.com');
  await browser.close();
});
```

#### 性能和优化

+   关于共享内存：

```javascript
Chrome 默认使用 /dev/shm 共享内存，但是 docker 默认/dev/shm 只有64MB，显然是不够使用的，提供两种方式来解决：
- 启动 docker 时添加参数 --shm-size=1gb 来增大 /dev/shm 共享内存，但是 swarm 目前不支持 shm-size 参数
- 启动 Chrome 添加参数 - disable-dev-shm-usage，禁止使用 /dev/shm 共享内存
```

+   尽量使用同一个浏览器实例，这样可以实现缓存共用
+   通过请求拦截没必要加载的资源
+   像我们自己打开 Chrome 一样，tab 页多必然会卡，所以必须有效控制 tab 页个数
+   一个 Chrome 实例启动时间长了难免会出现内存泄漏，页面奔溃等现象，所以定时重启 Chrome 实例是有必要的
+   为了加快性能，关闭没必要的配置，比如：-no-sandbox（沙箱功能），--disable-extensions（扩展程序）等
+   尽量避免使用 page.waifFor(1000)，让程序自己决定效果会更好
+   因为和 Chrome 实例连接时使用的 Websocket，会存在 Websocket sticky session 问题.

#### 参考文献

+   [初探 Headless Chrome](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27100187&source=article&objectId=1942245)
+   [Puppeteer 官方文档](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fpuppeteer%2Fpuppeteer&source=article&objectId=1942245)
+   [Puppeteer 指南](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fblog.xcatliu.com%2F2018%2F09%2F18%2Fpuppeteer_tutorial%2F&source=article&objectId=1942245)
+   [Puppeteer API](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fpuppeteer%2Fpuppeteer%2Fblob%2Fv5.4.1%2Fdocs%2Fapi.md%23&source=article&objectId=1942245)
+   [结合项目来谈谈 Puppeteer](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F76237595&source=article&objectId=1942245)
+   [Puppeteer性能优化与执行速度提升](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fblog.it2048.cn%2Farticle-puppeteer-speed-up%2F&source=article&objectId=1942245)