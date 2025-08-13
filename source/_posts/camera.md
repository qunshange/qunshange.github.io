---
title: 基于 uni-app 和腾讯地图 SDK 实现水印相机
date: 2025-08-07
excerpt: 实习期写的功能
tag: 
- 实际项目开发总结
- 水印相机
- canvas
- sdk
category:
- 前端
- 微信小程序
  
---

## 一、项目背景

微信小程序“水印相机” :用户拍照后，系统会自动在照片中加上当前的拍摄时间、详细的地理位置信息以及平台水印 Logo。

为了实现跨端复用，我们选择了 `uni-app` 框架进行开发。在定位功能实现上，微信原生的 `getLocation` 只能返回经纬度，因此我引入了腾讯地图 SDK，通过 `reverseGeocoder` 实现从坐标到地址的反解析，最终使用`canvas`打印地址时间信息,组合成可导出的带水印图片。

---

## 二、技术方案概览

- 使用 `uni.chooseImage` 进行拍照上传
- 用 `uni.getLocation` 获取 GPS 坐标
- 引入`微信小程序jsSDK`,接入 `qqmap-wx-jssdk` 获取用户地址（逆地理解析）
- 使用 `canvas` 绘制图片 + 多行文字 + 平铺 Logo 水印
- 用 `uni.canvasToTempFilePath` 导出图片 + 自动保存到相册

---

## 三、核心功能代码讲解
<P>前置操作 : 在腾讯位置服务的开发文档中找到申请密钥入口申请密钥,然后开通api服务,下载sdk并放在代码文件夹下的对应位置,在小程序管理后台中设置request合法域名;最后在代码中引入sdk核心类</P>

### 1. 获取当前定位（经纬度）

```js
async getLocation() {
  return new Promise((resolve, reject) => {
    uni.getLocation({
      type: 'gcj02',//中国加密坐标系
      success: res => resolve(res),
      fail: err => reject('未知地址')
    });
  });
}
```

### 2. 经纬度 → 地址：调用腾讯地图 SDK

```js
const geoRes = await new Promise((resolve, reject) => {
  qqmapsdk.reverseGeocoder({
    location: {
      latitude: locationRes.latitude,
      longitude: locationRes.longitude
    },
    success: res => resolve(res),
    fail: err => reject(err)
  });
});
```

### 3. 添加时间与地址水印到图片上

```js
ctx.setFontSize(fontSize);
ctx.setFillStyle('#FFFFFF');
ctx.fillText('xxxxxxx', 20, baseY + lineSpacing);
ctx.fillText(`经纬度: ${longitude}, ${latitude}`, 20, baseY);
ctx.fillText(addressText, 20, baseY - lineSpacing);
ctx.fillText(this.time, 20, baseY - 2 * lineSpacing);
```

### 4. 多图层 Logo 水印（淡化背景）

```js
ctx.setGlobalAlpha(0.2);
for (let i = 0; i < this.photoWidth; i += markW) {
  for (let j = 0; j < this.photoHeight; j += markH) {
    ctx.drawImage(this.appNameWatermark, i, j, markW, markH);
  }
}
```

### 5. 导出并保存图片到相册

```js
uni.canvasToTempFilePath({
  canvasId: 'canvasWatermark',
  success: (result) => {
    this.photoUrl = result.tempFilePath;
    uni.saveImageToPhotosAlbum({ filePath: this.photoUrl });
  }
});
```

---

## 四、个人思路整理

#### 1.点击事件
  按钮绑定点击事件,点击事件中 `chooseImage` 上传图片,限制数量和来源,在其中保存图片路径


```js
uni.chooseImage({
  count: 1, // 只选一张图片
  sourceType: ['camera'], // 限制来源为相机
  success: (res) => {
    this.photoUrl = res.tempFilePaths[0]; // 保存拍到的照片临时路径
```

#### 2.图片处理
  `uni.getImageInfo` 返回图片的宽高（像素单位），为后续 canvas 绘制提供准确的像素尺寸,保存到 this.photoWidth 和 this.photoHeight

  `uni.getSystemInfoSync().windowWidth:` 当前设备屏幕宽度（px）。
  
(图片宽度 / 屏幕宽度) * 750 → 把像素宽度转换为 rpx,同理计算高度的 rpx 值。

最后生成 canvas 的样式字符串（width: xxxrpx; height: xxxrpx;），让 canvas 在页面上等比例显示。
  调用 `addWatermarkToImage` 方法


```js
uni.getImageInfo({
  src: this.photoUrl,
  success: (info) => {
     this.photoWidth = info.width;
              this.photoHeight = info.height;
              const canvasWidthRpx = (info.width / uni.getSystemInfoSync().windowWidth) * 750;
              const canvasHeightRpx = (info.height /uni.getSystemInfoSync().windowWidth) * 750;
              this.canvasStyle = `width: ${canvasWidthRpx}rpx; height: ${canvasHeightRpx}rpx;`;
              this.addWatermarkToImage();
```
#### 3.添加水印初始化
在 `addWatermarkToImage` 方法中，首先将 `showCanvas` 设置为 `true` 并通过 `await this.$nextTick()` 等待视图更新，确保 `<canvas>` 元素已经渲染到页面上，然后通过 `uni.createCanvasContext('canvasWatermark', this)` 获取当前组件作用域下的画布上下文对象 `ctx`，用于后续绘图。接着调用 `this.getLocation()` 获取当前经纬度，并使用 `this.getTime()` 记录当前时间字符串。为了得到更直观的位置信息，通过腾讯地图 SDK 的 `reverseGeocoder` 接口将经纬度转换为具体的地址字符串，如果解析失败则使用 `'未知地址'` 作为兜底。
```js
 async addWatermarkToImage()
```
```js
      this.showCanvas = true;
      await this.$nextTick();
      const locationRes = await this.getLocation();
      this.getTime();

      let addressText = '未知地址';
      try {
        const geoRes = await new Promise((resolve, reject) => {
          qqmapsdk.reverseGeocoder({
            location: {
              latitude: locationRes.latitude,
              longitude: locationRes.longitude
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          });
        });
        addressText = geoRes.result.address;
      } catch (e) {
        console.warn('地址解析失败:', e);
      }

```
#### 4.添加水印
获取到定位与时间信息后，先调用 `ctx.drawImage(this.photoUrl, 0, 0, this.photoWidth, this.photoHeight)` 将原始照片绘制到画布上，然后根据图片宽度动态计算 `fontSize`、`lineSpacing` 和底部起始坐标 `baseY`，并将经纬度保留两位小数用于显示。为了绘制背景 Logo 水印，先将全局透明度设置为 0.2，通过 `uni.getImageInfo` 获取 Logo 图片尺寸，使用双层循环将 Logo 平铺绘制到整个画布区域。接着恢复透明度为 1.0，设置字体大小、颜色和对齐方式，从下往上依次绘制认证信息、经纬度、地址和时间四行文字水印。
```js
      const ctx = uni.createCanvasContext('canvasWatermark', this);
      ctx.drawImage(this.photoUrl, 0, 0, this.photoWidth, this.photoHeight);
      const fontSize = this.photoWidth / 25;
      const lineSpacing = fontSize + 10;
      const baseY = this.photoHeight - 20 - lineSpacing;
      const longitude = locationRes.longitude.toFixed(2);
      const latitude = locationRes.latitude.toFixed(2);

      ctx.setGlobalAlpha(0.2);
      uni.getImageInfo({
        src: this.appNameWatermark,
        success: (imgInfo) => {
          const markW = imgInfo.width;
          const markH = imgInfo.height;

          for (let i = 0; i < this.photoWidth; i += markW) {
            for (let j = 0; j < this.photoHeight; j += markH) {
              ctx.drawImage(this.appNameWatermark, i, j, markW, markH);
            }
          }

          ctx.setGlobalAlpha(1.0);
          ctx.setFontSize(fontSize);
          ctx.setFillStyle('#FFFFFF');
          ctx.setTextAlign('left');

          ctx.fillText('认证:xxxxxx', 20, baseY + lineSpacing);
          ctx.fillText(`经纬度: ${longitude}, ${latitude}`, 20, baseY);
          ctx.fillText(addressText, 20, baseY - lineSpacing);
          ctx.fillText(this.time, 20, baseY - 2 * lineSpacing);
```
#### 5.保存图片
绘制完成后，调用 `ctx.draw(true, callback)` 并在回调中使用 `uni.canvasToTempFilePath` 将带水印的画布导出为临时图片文件，并赋值给 `photoUrl` 更新页面显示，同时将 `showCanvas` 设为 `false` 隐藏画布。最后调用 `uni.saveImageToPhotosAlbum` 自动保存图片到系统相册，如果用户未授权保存权限则通过弹窗引导其前往设置界面开启权限。
```js
          ctx.draw(true, () => {
            uni.canvasToTempFilePath({
              canvasId: 'canvasWatermark',
              success: (result) => {
                this.photoUrl = result.tempFilePath;
                this.showCanvas = false;

                // 自动保存到系统相册
                uni.saveImageToPhotosAlbum({
                  filePath: this.photoUrl,
                  success: () => {
                    uni.showToast({
                      title: '已保存到相册',
                      icon: 'success'
                    });
                  },
                  fail: (err) => {
                    if (err.errMsg.includes('auth deny')) {
                      uni.showModal({
                        title: '提示',
                        content: '需要授权保存相册权限',
                        success(res) {
                          if (res.confirm) {
                            uni.openSetting();
                          }
                        }
                      });
                    } else {
                      uni.showToast({
                        title: '保存失败',
                        icon: 'none'
                      });
                    }
                  }
                });
              },
              fail: (error) => {
                console.error('图片导出失败:', error);
              }
            });
          });
```

---


> 如果你对 `qqmap-wx-jssdk` 的其他 API 感兴趣，也欢迎查阅：[腾讯地图小程序开发文档](https://lbs.qq.com/miniProgram/jsSdk/jsSdkGuide/jsSdkOverview)
