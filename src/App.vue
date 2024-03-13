<template>
  <div>
    <h1>视频上传或录制</h1>
    <!-- 视频上传 -->
    <input type="file" accept="video/*" @change="handleFileChange" />
    <!-- 摄像头录制视频 -->
    <button @click="startRecording">开始录制</button>
    <button @click="stopRecording" :disabled="!isRecording">停止录制</button>
    <!--实时预览-->
    <video ref="videoPreview" autoplay muted></video>
    <video ref="videoPlayback" controls></video>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isRecording: false,
      mediaRecorder: null,
      recordedChunks: [],
      stream: null,
    };
  },
  methods: {
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.uploadVideo(file);
      }
    },
    async uploadVideo(blob) {
      try {
        const formData = new FormData();
        formData.append('video', blob, 'video.webm');

        const response = await fetch('http://localhost:8000/video/test', {
          method: 'POST',
          body: formData,
        });
        if (response.ok) {
          alert('上传成功');
          this.$refs.videoPlayback.src = URL.createObjectURL(response.data);
        } else {
          alert('上传失败');
        }
      } catch (err) {
        console.error('上传失败', err);
      }
    },
    async startRecording() {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        try {
          this.stream = await navigator.mediaDevices.getUserMedia({ video: true });
          this.$refs.videoPreview.srcObject = this.stream;
          this.mediaRecorder = new MediaRecorder(this.stream);
          this.mediaRecorder.ondataavailable = event => {
            if (event.data.size > 0) {
              this.recordedChunks.push(event.data);
            }
          };
          this.mediaRecorder.start(10); // 每10ms记录一次数据
          this.isRecording = true;
        } catch (err) {
          console.error("无法获取媒体设备", err);
        }
      }
    },
    stopRecording() {
      if (this.mediaRecorder) {
        this.mediaRecorder.stop();
        this.isRecording = false;
        this.createVideoPlayback();
        this.stream.getTracks().forEach(track => track.stop());
      }
    },
    createVideoPlayback() {
      const blob = new Blob(this.recordedChunks, { type: 'video/webm' });
      this.recordedChunks = [];
      this.uploadVideo(blob);
    },
  },
};
</script>
