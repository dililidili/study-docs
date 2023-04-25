## 1.2、Vue用vue-qecode-reader实现扫一扫功能

```
<!-- QrcodeReader.vue  下方案例为如何使用-->
<template>
  <div class="qrcode">
    <div class="code"  v-show="qrcode">
      <!-- decode是扫描结果的函数，torch用于是否需要打开手电筒，init用于检查该设备是否能够调用摄像头的权限，camera可用于打开前面或者后面摄像头  -->
      <div class="code-button">
        <button @click="switchCamera">相机反转</button>
        <button @click="turnCameraOff">关闭相机</button>
      </div>
<!--      camera:相机开启关闭 torchActive：闪光灯开启关闭 -->
      <qrcode-stream
          :camera="camera"
          :torch="torchActive"
          @decode="onDecode"
          @init="onInit"
      >
        <div>
          <div class="qr-scanner">
            <div class="box">
              <div class="line"></div>
              <div class="angle"></div>
            </div>
            <div class="txt">
              将二维码/条码放入框内，即自动扫描
              <div class="myQrcode">我的二维码</div>
            </div>
          </div>
        </div>
      </qrcode-stream>
    </div>
  </div>
</template>

<script>
// 引用vue-qrcode-reader插件
import { QrcodeStream} from 'vue-qrcode-reader'

export default {
  name: 'Approve',
  props: {
  },
  data() {
    return {
      qrcode:false,
      camera:'off',
      torchActive:false,
    }
  },
  created() {},
  //销毁页面时，关闭相机
  destroyed() {
    this.qrcode=false;
    this.camera='off'
  },

  components: {
    // 注册组件
    QrcodeStream,
  },
  methods: {
    // 扫码结果回调
    onDecode(result) {
      this.$emit('onDecode', result)
    },
    // 相机反转  front前置 rear后置  auto自动识别
    switchCamera() {
      switch (this.camera) {
        case 'front':
          this.camera = 'rear'
          break
        case 'rear':
          this.camera = 'front'
          break
        default:
          this.camera = 'rear'
          break
      }
    },
    //开启摄像头
    turnCameraOn() {
      this.qrcode = true
      // camera:: 'rear'--前摄像头，'front'后摄像头，'off'关闭摄像头，会获取最后一帧显示，'auto'开始获取摄像头
      this.camera = 'auto'
    },
    // 关闭摄像头
    turnCameraOff() {
      this.camera = 'off'
      this.qrcode = false
      this.$emit('qrcodeShowFalse')
    },
    // 打开手电筒
    ClickFlash() {
      switch (this.torchActive) {
        case true:
          this.torchActive = false
          break
        case false:
          this.torchActive = true
          break
      }
    },
    // 检查是否调用摄像头
    async onInit(promise) {
      try {
        await promise
      } catch (error) {
        let errorMessage="";
        if (error.name === 'NotAllowedError') {
          errorMessage = 'ERROR: 您需要授予相机访问权限';
        } else if (error.name === 'NotFoundError') {
          errorMessage = 'ERROR: 这个设备上没有摄像头';
        } else if (error.name === 'NotSupportedError') {
          errorMessage = 'ERROR: 所需的安全上下文(HTTPS、本地主机)';
        } else if (error.name === 'NotReadableError') {
          errorMessage = 'ERROR: 相机被占用';
        } else if (error.name === 'OverconstrainedError') {
          errorMessage = 'ERROR: 安装摄像头不合适';
        } else if (error.name === 'StreamApiNotSupportedError') {
          errorMessage = 'ERROR: 此浏览器不支持流API';
        }
        alert(errorMessage)
      }
    },
  },
}
</script>
<style scoped>

/*让整个摄像页面被蓝色格子笼罩*/
.qr-scanner {
  background-image: linear-gradient(
      0deg,
      transparent 24%,
      rgba(32, 255, 77, 0.1) 25%,
      rgba(32, 255, 77, 0.1) 26%,
      transparent 27%,
      transparent 74%,
      rgba(32, 255, 77, 0.1) 75%,
      rgba(32, 255, 77, 0.1) 76%,
      transparent 77%,
      transparent
  ),
  linear-gradient(
      90deg,
      transparent 24%,
      rgba(32, 255, 77, 0.1) 25%,
      rgba(32, 255, 77, 0.1) 26%,
      transparent 27%,
      transparent 74%,
      rgba(32, 255, 77, 0.1) 75%,
      rgba(32, 255, 77, 0.1) 76%,
      transparent 77%,
      transparent
  );
  background-size: 3rem 3rem;
  background-position: -1rem -1rem;
  width: 100%;
  height: 666px;
  position: relative;
  background-color: #1110;
}

 /*摄像头大小和位置  宽高需要趋近于1:1*/
 .qrcode-stream-wrapper >>> .qrcode-stream-camera {
   margin: 0 auto;
   width: 520px;
  height: 500px;
  clear: both;
  margin-top: 20px;
}
/*中间用来放二维码的 四角框*/
.qr-scanner .box {
  width: 213px;
  height: 213px;
  position: absolute;
  left: 50%;
  top: 30%;
  transform: translate(-50%, -50%);
  overflow: hidden;
  border: 0.1rem solid rgba(0, 255, 51, 0.2);
}
/*扫描二维码提示语*/
.qr-scanner .txt {
  width: 100%;
  height: 35px;
  line-height: 35px;
  font-size: 14px;
  text-align: center;
  margin: 0 auto;
  position: absolute;
  top: 50%;
  left: 0;
}
.qr-scanner .myQrcode {
  text-align: center;
  color: #00ae10;
}

/*二维码框内来回晃动的线  动感扫描线*/
.qr-scanner .line {
  height: calc(100% - 2px);
  width: 100%;
  background: linear-gradient(180deg, rgba(0, 255, 51, 0) 43%, #00ff33 211%);
  border-bottom: 3px solid #00ff33;
  transform: translateY(-100%);
  animation: radar-beam 2s infinite alternate;
  animation-timing-function: cubic-bezier(0.53, 0, 0.43, 0.99);
  animation-delay: 1.4s;
}

.qr-scanner .box:after,
.qr-scanner .box:before,
.qr-scanner .angle:after,
.qr-scanner .angle:before {
  content: '';
  display: block;
  position: absolute;
  width: 3vw;
  height: 3vw;

  border: 0.2rem solid transparent;
}

.qr-scanner .box:after,
.qr-scanner .box:before {
  top: 0;
  border-top-color: #00ff33;
}

.qr-scanner .angle:after,
.qr-scanner .angle:before {
  bottom: 0;
  border-bottom-color: #00ff33;
}

.qr-scanner .box:before,
.qr-scanner .angle:before {
  left: 0;
  border-left-color: #00ff33;
}

.qr-scanner .box:after,
.qr-scanner .angle:after {
  right: 0;
  border-right-color: #00ff33;
}

@keyframes radar-beam {
  0% {
    transform: translateY(-100%);
  }

  100% {
    transform: translateY(0);
  }
}
</style>










<!--
案例  v-show="qrcodeShow"为非必要代码

<template>
  <div class="qrcode">
    <button @click="clickCode">打开相机</button>
    <qrcode
        ref="qrcode"
        v-show="qrcodeShow"
        @qrcodeShowFalse="qrcodeShowFalse"
        @onDecode="onDecode"
    />
  </div>
</template>
<script>
export default {
  data() {
    return {
      qrcodeShow:false,
    }
  },
  mounted() {},
  methods: {
    // 打开相机
    clickCode() {
     this.$refs.qrcode.turnCameraOn();
     this.qrcodeShow=true;
    },
    // 扫码结果回调
    onDecode(result) {
      alert(result)
      this.qrcodeShow=false;
      this.$refs.qrcode.turnCameraOff()
    },
    qrcodeShowFalse(){
      this.qrcodeShow=false;
    },


  },
  components: {
    // 注册
    qrcode: () => import('@/components/util/QrcodeReader'),
  },
}
</script>




-->
```



**注意：vue-qrcode-reader这个插件只适用于本地调试（localhost）或者带有https的域名，http是不能用这个插件的（可能调用摄像头存在隐私问题）。具体原因请查看官方文档**[官方文档](https://gruhn.github.io/vue-qrcode-reader/demos/Simple.html)

解决办法 在vue框架中配置HTTPS的运行环境

在vue.config.js中进行如下配置：

```
// 配置反向代理
  devServer: {
    proxy: {
      '/api': {
        target: 'https://172.31.120.61:8080/',
        ws: true,
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
    // 开启https 访问时使用https://172.31.120.61:8081 
    https: true
  }
```

