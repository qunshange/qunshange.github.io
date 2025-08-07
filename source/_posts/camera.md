---
title: 基于 uni-app 和腾讯地图 SDK 实现水印相机
date: 2025-08-07
tag: 
- 实际项目开发总结
- 水印相机
category:
- 微信小程序
- 前端
---

## 一、项目背景

在我参与的实习项目中，有一个功能需求是实现一款微信小程序“水印相机”。用户拍照后，系统会自动在照片中加上当前的拍摄时间、详细的地理位置信息以及平台水印 Logo。

为了实现跨端复用，我们选择了 `uni-app` 框架进行开发。在定位功能实现上，微信原生的 `getLocation` 只能返回经纬度，因此我们引入了腾讯地图 SDK，通过 `reverseGeocoder` 实现从坐标到地址的反解析，最终组合成可导出的带水印图片。

---

## 二、技术方案概览

- 使用 `uni.chooseImage` 进行拍照上传
- 用 `uni.getLocation` 获取 GPS 坐标
- 接入 `qqmap-wx-jssdk` 获取用户地址（逆地理解析）
- 使用 `canvas` 绘制图片 + 多行文字 + 平铺 Logo 水印
- 用 `uni.canvasToTempFilePath` 导出图片 + 自动保存到相册

---

## 三、核心功能代码讲解

### 1. 获取当前定位（经纬度）

```js
async getLocation() {
  return new Promise((resolve, reject) => {
    uni.getLocation({
      type: 'gcj02',
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
ctx.fillText('认证:装饰装修保障服务平台', 20, baseY + lineSpacing);
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

## 四、常见坑及注意事项

- `canvas-id` 与 `canvasId` 要对应，注意大小写
- `draw()` 后必须等回调后再执行 `canvasToTempFilePath`
- iOS 下图片宽高与 canvas 绘图比例可能不一致
- 使用 `saveImageToPhotosAlbum` 前需用户授权

---

## 五、总结与扩展

本次功能实现让我熟悉了 `uni-app` 在微信小程序平台的能力边界，同时也理解了引入第三方 SDK（如腾讯地图）的实际价值。

如果后续要拓展，还可以支持：

- 拍摄视频并添加水印
- 自定义水印模板内容（如打卡名称、项目编号）
- 上传至云存储平台（如腾讯云 COS、阿里云 OSS）

---

> 如果你对 `qqmap-wx-jssdk` 的其他 API 感兴趣，也欢迎查阅：[腾讯地图小程序开发文档](https://lbs.qq.com/miniProgram/jsSdk)
