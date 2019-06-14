<template>
  <div class="container" id="home">
    <div id="appInner">
      <!-- built files will be auto injected -->
      <div class="container" id="main-area">
        <form ref="vExample">
          <input type="file" style="display:none;" ref="vExampleFile" @change="vExampleUpload">
        </form>
        <div class="row" style="padding:10px;">
          <h4>示例1：腾讯云直接上传视频</h4>
          <a href="javascript:void(0);" class="btn btn-default" @click="vExampleAdd">直接上传视频</a>
        </div>

        <div class="row" id="resultBox">
          <!-- 上传信息组件	 -->
          <div class="uploaderMsgBox" v-for="uploaderInfo in uploaderInfos">
            <div v-if="uploaderInfo.videoInfo">
              视频名称：{{uploaderInfo.videoInfo.name + '.' + uploaderInfo.videoInfo.type}}；
              上传进度：{{Math.floor(uploaderInfo.progress * 100) + '%'}}；
              上传速度：{{Math.floor(uploaderInfo.speed / 1024) + 'KB/s'}}；
              fileId：{{uploaderInfo.fileId}}；
              上传结果：{{uploaderInfo.isVideoUploadCancel ? '已取消' : uploaderInfo.isVideoUploadSuccess ? '上传成功' : '上传中'}}；
              <br>
              地址：{{uploaderInfo.videoUrl}}；
              <a
                href="javascript:void(0);"
                class="cancel-upload"
                v-if="!uploaderInfo.isVideoUploadSuccess && !uploaderInfo.isVideoUploadCancel"
                @click="uploaderInfo.cancel()"
              >取消上传</a>
              <br>
            </div>
          </div>
        </div>
      </div>

      <div class="container" id="main-area">
        <form ref="vExample">
          <input
            type="file"
            style="display:none;"
            ref="vExampleFilealiyun"
            @change="vExampleUploadaliyun"
          >
        </form>
        <div class="row" style="padding:10px;">
          <h4>示例1：阿里云直接上传视频</h4>
          <a href="javascript:void(0);" class="btn btn-default" @click="vExampleAddaliyun">直接上传视频</a>
          <a href="javascript:void(0);" class="btn btn-default" @click="uploadAliyunLog"> 上传日志</a>
        </div>

        <div class="row" id="resultBox">
          <!-- 上传信息组件	 -->
          <div class="uploaderMsgBox">
            上传进度：{{aliyunprogress}}
            上传结果：{{aliyunResult === 1 ? '上传成功':aliyunResult === 0 ? '上传中':aliyunResult === 2 ?'上传失败':aliyunResult === 3 ?'取消上传':''}}
            <br>
            <a href="javascript:void(0);" class="cancel-upload" @click="cancelaliyunUpload">取消上传</a>
            <br>
            <br>
            <a href="javascript:void(0);" class="cancel-upload" @click="resumealiyunUpload">恢复上传</a>
            <br>
            <br>
            <a href="javascript:void(0);" class="cancel-upload" @click="initClient">获取token</a>
            <br>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Logline from 'logline';
import qs from 'qs';
export default {
  name: "home",
  components: {},
  data: function() {
    return {
      uploaderInfos: [],
      aliyunuploaderInfos: [],
      aliyunprogress: 0,
      aliyunResult: -1,
      client: null,
      aliyunResumeCheckPoint: {},
      sdkLog: null,
      appServer: "http://test.lyy.life:8080/",
      bucket: "test-art-h5",
      region: "oss-cn-hangzhou"
    };
  },
  created: function() {
    this.tcVod = new TcVod.default({
      getSignature: this.getSignature
    });
  },
  mounted() {
    Logline.using(Logline.PROTOCOL.INDEXEDDB)
    this.sdkLog = new Logline('sdk');
  },
  methods: {
    getSignature() {
      return axios
        .get("http://test.lyy.life:8080/uploadSignature", JSON.stringify({}))
        .then(function(response) {
          console.log(response.data);
          return response.data;
        });
    },
    /**
     * vExample示例。添加视频
     **/
    vExampleAdd: function() {
      console.log(`vExampleAdd`);
      this.$refs.vExampleFile.click();
    },
    vExampleAddaliyun: function() {
      console.log(`vExampleAddaliyun`);
      this.$refs.vExampleFilealiyun.click();
    },
    /**
     * vExample示例。上传视频过程。
     **/
    vExampleUpload: function() {
      var self = this;
      var videoFile = this.$refs.vExampleFile.files[0];
      console.log("123");
      var uploader = this.tcVod.upload({
        videoFile: videoFile
      });
      uploader.on("video_progress", function(info) {
        uploaderInfo.progress = info.percent;
        uploaderInfo.speed = info.speed;
        console.log(info);
      });
      uploader.on("video_upload", function(info) {
        uploaderInfo.isVideoUploadSuccess = true;
      });
      console.log(uploader, "uploader");
      var uploaderInfo = {
        videoInfo: uploader.videoInfo,
        isVideoUploadSuccess: false,
        isVideoUploadCancel: false,
        progress: 0,
        fileId: "",
        videoUrl: "",
        cancel: function() {
          uploaderInfo.isVideoUploadCancel = true;
          uploader.cancel();
        }
      };
      this.uploaderInfos.push(uploaderInfo);
      uploader
        .done()
        .then(function(doneResult) {
          console.log("doneResult", doneResult);
          uploaderInfo.fileId = doneResult.fileId;
        })
        .then(function(videoUrl) {
          uploaderInfo.videoUrl = videoUrl;
          self.$refs.vExample.reset();
        });
    },
    cancelaliyunUpload: function() {
      this.client.cancel();
    },
    resumealiyunUpload: function() {
      var self = this;
      var videoFile = this.$refs.vExampleFilealiyun.files[0];
      let progress = function(p, checkpoint, result) {
        self.aliyunResumeCheckPoint = checkpoint;
        self.aliyunprogress = Math.floor(p * 100) + "%";
        console.log(
          "+++++++++" +
            Math.floor(p * 100) +
            "%" +
            "\ncheckpoint:" +
            JSON.stringify(checkpoint) +
            "\n\nresult" +
            JSON.stringify(result)
        );
      };
      this.client
        .multipartUpload("videoFileName", videoFile, {
          progress: progress,
          checkpoint: self.aliyunResumeCheckPoint
        })
        .then(res => {
          console.log("upload success: %j", res.toString());
          self.aliyunResult = 1;
        })
        .catch(err => {
          console.log("err:" + err.toString());
          if (err.name === "cancel") {
            self.aliyunResult = 3;
          } else {
            self.aliyunResult = 2;
          }
        });
    },
    initClient() {
      var self = this;
      var url = this.appServer + "stsToken";
      axios.get(url, JSON.stringify({})).then(function(response) {
        var creds = response.data;
        console.log(creds);
        self.client = new OSS({
          endpoint: "testh5.yishihui.com",
          cname: true,
          region: self.region,
          accessKeyId: creds.AccessKeyId,
          accessKeySecret: creds.AccessKeySecret,
          stsToken: creds.SecurityToken,
          bucket: self.bucket
        });
        self.collectLog()
      });
    },
    collectLog: function() {
      const _debug = OSS.prototype.debug;
      OSS.prototype.debug = (...params) => {
        console.log('collectLog');
        console.log(...params);
        console.log('collectLog');
        // 获取params最后一个参数的数值是info还是error，如果是info就调用sdkLog.info error的话就调用 sdkLog.error
        const info = params[params.length - 1];
        // 解析params参数 ["request %s %s, with headers %j, !!stream: %s", "GET", "http://luozhang002.oss-cn-zhangjiakou.aliyuncs.com/?max-keys=100", {…}, false, "info"]
        // 规则是 %s 或者%s, 替换 字符, %j 或者%j ,替换为对象, 或者把%j对象单独打印出来
        // 解析逻辑 (开发者也可以自己写，这里先提供简单的解析代码，可供参考)
        // 最终要存储的总的信息 params.length的长度约定的至少为2，为2 直接取第一个，其他的长度需要另外处理
        let resultInfo = "";
        if (params.length === 2) {
          resultInfo = params[0];
        } else {
          let k = 1;
          const firstInfo = params[0].split(" ");
          const newInfo = [];
          for (let j = 0; j < firstInfo.length; j++) {
            if (firstInfo[j] === "%s" || firstInfo[j] === "%s,") {
              newInfo.push(params[k]);
              k++;
            } else if (firstInfo[j] === "%j" || firstInfo[j] === "%j,") {
              newInfo.push(JSON.stringify(params[k]));
              k++;
            } else {
              newInfo.push(firstInfo[j]);
            }
          }
          resultInfo = newInfo.join(" ");
        } // end else
        if (info === "info") {
          this.sdkLog.info("info", resultInfo);
        } else {
          this.sdkLog.error("error", resultInfo);
        }
        // 之前debug的逻辑还是保留的，localstoreage.debug = 'ali-oss'
        _debug(...params);
      };
    },
    vExampleUploadaliyun: function() {
      this.aliyunResult = 0;
      console.log(`vExampleUploadaliyun`);
      var self = this;
      var videoFile = this.$refs.vExampleFilealiyun.files[0];
      var url = this.appServer + "stsToken";
      axios.get(url, JSON.stringify({})).then(function(response) {
        var creds = response.data;
        console.log(creds);

        self.client = new OSS({
          endpoint: "testh5.yishihui.com",
          cname: true,
          region: self.region,
          accessKeyId: creds.AccessKeyId,
          accessKeySecret: creds.AccessKeySecret,
          stsToken: creds.SecurityToken,
          bucket: self.bucket
        });
        let progress = function(p, checkpoint, result) {
          self.aliyunResumeCheckPoint = checkpoint;
          self.aliyunprogress = Math.floor(p * 100) + "%";
          console.log(
            "+++++++++" +
              Math.floor(p * 100) +
              "%" +
              "\ncheckpoint:" +
              JSON.stringify(checkpoint) +
              "\n\nresult" +
              JSON.stringify(result)
          );
        };
        // const options = {
        //   progress,
        //   partSize: 100 * 1024,
        // };
        self.collectLog()
        self.client.multipartUpload(creds.FileName, videoFile, { progress: progress })
          .then(res => {
            console.log("upload success: ", res.toString());
            self.aliyunResult = 1;
          })
          .catch(err => {
            console.log("err:" + err.toString());
            if (err.name === "cancel") {
              self.aliyunResult = 3;
            } else {
              self.aliyunResult = 2;
            }
          });
      });
    },
    sendLog(logs){
      if (logs.length == 0) {
        return
      }
      var that = this
      axios.post('https://videotest.yishihui.com/commonapi/statistics/uploadLogFromFrontend',qs.stringify({
          logType: '40',
          eventId: '899998',
          extParams:JSON.stringify({'msg':logs[0]})
          }))
          .then(function (response) {
          console.log(response);
          logs.shift()
          that.sendLog(logs)
          })
          .catch(function (error) {
          console.log(error);
          that.sendLog(logs)
          });
    },
    uploadAliyunLog() {
      var that = this
      Logline.all(function(logs) {
        // process logs here这里可以获取到所有的日志是一个大数组，用户可以自行把数据上传到任何地方OSS、STS或者其他日志分析系统
        console.log(logs);
        that.sendLog(logs)
      });
    }
  }
};
</script>

<style>
.text-danger {
  color: red;
}
.control-label {
  text-align: left !important;
}
#resultBox {
  width: 100%;
  height: 300px;
  border: 1px solid #888;
  padding: 5px;
  overflow: auto;
  margin-bottom: 20px;
}
.uploaderMsgBox {
  width: 100%;
  border-bottom: 1px solid #888;
}
.cancel-upload {
  text-decoration: none;
  cursor: pointer;
}
</style>
