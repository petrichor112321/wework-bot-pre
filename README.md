# Wework Bot
[![Powered by Wechaty](https://img.shields.io/badge/Powered%20By-Wechaty-green.svg)](https://wechaty.js.org)
[![Wechaty Contributor Program](https://img.shields.io/badge/Wechaty-Contributor%20Program-green.svg)](https://wechaty.js.org/docs/contributor-program)
[![Juzi.BOT Developer Program](https://img.shields.io/badge/Wechaty%Contributor%20Program-Juzi.BOT-orange.svg)](https://github.com/juzibot/Welcome/wiki/Everything-about-Wechaty/)  

企业微信机器人，企微扫码登录(可在外部群使用，
`非webhook、群助理等`)  

如此一来限制更少，功能更多。  
本项目基于开源项目[ 一个 基于 chatgpt + wechaty 的微信机器人]([链接地址](https://github.com/wangrongding/wechat-bot))的微信机器人二次开发而来，很优秀的项目，可以去看看。
但协议不同，所以改动也蛮多的。

可以用来帮助你自动回复企微消息，或者管理微信群/好友.
## 声明：  `需要第三方token，付费版本，白嫖勿扰！！` 
### 过程中遇到很多问题，都是看issues解决的，所以算是问题汇总。
使用本仓库代码只需对应位置填入你的token即可使用！！本仓库属于二次开发，已有openai，kimi俩种ai接入方式。
## 问题汇总

### 问题1：[第一次登录需要监听与输入企业微信登录验证码]([链接地址]())
[https://github.com/wechaty/puppet-supports/issues/267]([链接地址]https://github.com/wechaty/puppet-supports/issues/267) 

 <img src="https://b2.kuibu.net/file/imgdisk/2024/04/02/-2024-04-02-141857.png"  width="200"> 



确实给到了很详细的方法，示例项目是ts版本的，我用了js实现  
注：首先注意依赖版本，创建示例时也要注意引入的是`@juzi/wechaty`  
对应类型 ：  
VerifyCodeStatus =1 &nbsp;&nbsp;&nbsp;&nbsp; 待输入状态   
VerifyCodeScene  =1 &nbsp;&nbsp;&nbsp;&nbsp; 验证场景（目前只有登录场景）  
### 相关代码：
```javascript
// 验证码
const store = {
  qrcodeKey: '',
}
// 扫码
bot.on('scan', onScan)
// 验证码
bot.on('verify-code', onVerify) 

function onScan(qrcode, status) {
  if (status === ScanStatus.Waiting || status === ScanStatus.Timeout) {
    // 在控制台显示二维码
    store.qrcodeKey = getQrcodeKey(qrcode) || ''
    qrTerminal.generate(qrcode, { small: true })
    const qrcodeImageUrl = ['https://api.qrserver.com/v1/create-qr-code/?data=', encodeURIComponent(qrcode)].join('')
    console.log('onScan:', qrcodeImageUrl, ScanStatus[status], status)
  } else {
    log.info('onScan: %s(%s)', ScanStatus[status], status)
  }
}
async function onVerify(id, message, scene, status) {

  // 需要注意的是，验证码事件不是完全即时的，可能有最多10秒的延迟。
  // 这与底层轮询二维码状态的时间间隔有关。
  if (status === 1 && scene === 1 && id === store.qrcodeKey) {
    console.log(`receive verify-code event, id: ${id}, message: ${message}, scene: ${scene} status: ${status}`)
    // const verifyCode = '058948' // 通过一些途径输入验证码
    const fileContent = fs.readFileSync(filePath, 'utf8')
    const verifyCode = fileContent 
    try {
      await bot.enterVerifyCode(id, verifyCode) // 如果没抛错，则说明输入成功，会推送登录事件
      return
    } catch (e) {
      console.log(e.message)
      // 如果抛错，请根据 message 处理，目前发现可以输错3次，超过3次错误需要重新扫码。
      // 错误关键词: 验证码错误输入错误，请重新输入
      // 错误关键词：验证码错误次数超过阈值，请重新扫码'
      // 目前不会推送 EXPIRED 事件，需要根据错误内容判断
    }
  }
}
```
然后我就去扫码了，发现不可能一开始就把验证码写好啊，好像也没用看到获取验证码的方法，于是看到了`通过一些途径输入验证码`这个关键字眼，再加上有轮询，所以我就想到了根目录新建一个`.txt`文件，手机上显示后把验证码写进去，然后轮询不就读到了吗（有别的实现方法也可以给我提pr哈）
### 相关代码：
```javascript
// 验证码
import fs from 'fs'
import path from 'path'
// 获取项目根目录路径
const rootDir = process.cwd()
// 根目录下的文本文件名
const fileName = 'code.txt'
// 构建文件的完整路径
const filePath = path.resolve(rootDir, fileName)
//然后写到`通过一些途径输入验证码`也就是onVerify方法中 的部分就ok了
const fileContent = fs.readFileSync(filePath, 'utf8')
const verifyCode = fileContent // 通过一些途径输入验证码
```
第一次验证过后也就不需要再验证了

### 问题二：[Cannot use 'in'operator to search for 'port'in undefined]([链接地址]())  
[https://github.com/wechaty/puppet-supports/issues/267]([链接地址]https://github.com/wechaty/puppet-supports/issues/364#issuecomment-1952270004) 

 <img src="https://b2.kuibu.net/file/imgdisk/2024/04/02/-2024-04-02-145111.png" > 

这个官方也已经有解答，是依赖grpc中的错误，可以通过：  
修改 node_modules/@grpc/grpc-js/build/src/load-balancer-pick-first.js的411行

从
```javascript
const rawAddressList = [].concat(...endpointList.map(endpoint => endpoint.addresses));
改为
const rawAddressList = endpointList;
```


### 问题三：[token验证问题WorkPro:在win设置环境变量后启动还是报错，提示token不存在]([链接地址](https://github.com/wangrongding/wechat-bot))  
[https://github.com/wechaty/puppet-supports/issues/267]([链接地址]https://github.com/wechaty/puppet-supports/issues/364#issuecomment-1952270004) 

 <img src="https://b2.kuibu.net/file/imgdisk/2024/04/02/-2024-04-02-145838.png" > 
 可以先通过这个链接去测试token是否有效

 https://token-service-discovery-test.juzibot.com/v0/hosties/YOUR_TOKEN
 如果有效，那就是环境变量设置的问题，我设置了  
 WECHATY_PUPPET_SERVICE_AUTHORITY=token-service-discovery-test.juzibot.com
还是不行，然后我在实例化的时候加入了这个配置，这个在token领取的页面也有提到，他们社区服务器不稳定，会出现查询不存在的情况，换成他们自己的就好了。
```javascript
const WECHATY_PUPPET_SERVICE_TOKEN = '' // Replace with your purchased token
export const bot = WechatyBuilder.build({
  name: 'WechatEveryDay',
  puppet: '@juzi/wechaty-puppet-service', 
  puppetOptions: {
    token: WECHATY_PUPPET_SERVICE_TOKEN,
    authority: 'token-service-discovery-test.juzibot.com',//修稿验证服务器
    tls: {
      disable: true,
    },
  },
})
```
# 效果演示
`如果对您有所帮助，请点个 Star ⭐️ 支持一下`。


 <img src="https://b2.kuibu.net/file/imgdisk/2024/04/01/-2024-04-01-155750.png"> 

  ## 可以看到主体是真实用户，而不是自建应用等
<div align="center">

  <img src="https://b2.kuibu.net/file/imgdisk/2024/04/01/309654223aaa22186e408c19d8c2fa1.jpg" width="200">
</div>

# 联系方式
<div align="center">
 <img src="https://b2.kuibu.net/file/imgdisk/2024/04/01/797384744706512f007454ea969e06e288e3d7e0fe7b115.jpg" width="200px"> 
 </div>