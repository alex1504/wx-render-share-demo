## 微信小程序生成二维码分享图片示例

canvas画图片，画文字，使用clip方法裁剪圆形图片

![result](https://raw.githubusercontent.com/alex1504/wx-render-share-demo/master/assets/result.jpg)

```
// index.js
Page({
    data: {
        imagePath: "/images/share_img.png",
        bgPath: "/images/bg.png",
        avatarPath: "/images/avatar.png",
        qrcodePath: "/images/qrcode.png",
        maskHidden: true,
    },
    onLoad: function (options) {
        this.setCanvasSize();
    },
    //适配不同屏幕大小的canvas，生成的分享图宽高分别是750和940
    setCanvasSize: function () {
        const size = {};
        try {
            const res = wx.getSystemInfoSync();
            const scale = 750; //画布宽度
            const scaleH = 940 / 750; //生成图片的宽高比例
            const width = res.windowWidth; //画布宽度
            const height = res.windowWidth * scaleH; //画布的高度
            size.w = width;
            size.h = height;
        } catch (e) {
            // Do something when catch error
            console.log("获取设备信息失败" + e);
        }
        return size;
    },
    settextSec: function (context) {
        let that = this;
        const size = that.setCanvasSize();
        const textSec = "D2发起了一个群活动";
        const textSec1 = "长按二维码识别立即参与";
        context.setFontSize(14);
        context.setTextAlign("center");
        context.setFillStyle("#fefefe");
        context.fillText(textSec, size.w / 2, size.h * 0.88);
        context.fillText(textSec1, size.w / 2, size.h * 0.93);
        context.stroke();
    },
    drawCircleImg: function (ctx, img, x, y, r) {
        ctx.save();
        var d = 2 * r;
        var cx = x - r;
        var cy = y - r;
        ctx.beginPath();
        ctx.arc(x, y, r, 0, 2 * Math.PI);
        ctx.clip();
        ctx.drawImage(img, cx, cy, d, d);
        ctx.restore();
    },
    createNewImg: function () {
        const that = this;
        const size = that.setCanvasSize();
        const context = wx.createCanvasContext('myCanvas');
        const bgPath = this.data.bgPath;
        const qrcodePath = that.data.qrcodePath;
        const avatarPath = this.data.avatarPath;
        context.drawImage(bgPath, 0, 0, size.w, size.h);

        this.drawCircleImg(context, qrcodePath, size.w / 2, size.h * 0.32, size.w * 0.23)
        // context.drawImage(qrcodePath, size.w / 2 - 60, size.h * 0.32, size.w * 0.33, size.w * 0.33);

        context.drawImage(avatarPath, size.w / 2 - 25, size.h * 0.7, size.w * 0.14, size.w * 0.14);
        this.settextSec(context);
        console.log(size.w, size.h)
        //绘制图片
        context.draw();
        //将生成好的图片保存到本地，需要延迟一会，绘制期间耗时
        wx.showToast({
            title: '生成中...',
            icon: 'loading',
            duration: 2000
        });
        setTimeout(function () {
            wx.canvasToTempFilePath({
                canvasId: 'myCanvas',
                success: function (res) {
                    const tempFilePath = res.tempFilePath;
                    console.log(tempFilePath);
                    that.setData({
                        imagePath: tempFilePath,
                        canvasHidden: false,
                        maskHidden: true,
                    });

                    //预览图片
                    const img = that.data.imagePath;
                    wx.previewImage({
                        current: img,
                        urls: [img]
                    })

                },
                fail: function (res) {
                    console.log(res);
                }
            });
        }, 2000);
    },
    renderShareImg() {
        this.createNewImg()
    }
})
```
