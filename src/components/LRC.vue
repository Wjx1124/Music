<template>
  <div class="lrcContainer">
    <div class="lrc">
    {{ currentTimes }} - {{ durationTime }}
      <p v-for="(value,key,index) in lrcData" :key="index">{{ value }}</p>
    </div>
  </div>
</template>
<script>
export default {
  name: "LRC",
  data() {
    return {
      lrcData: {}
    };
  },
  props: {
    id: {
      type: [String, Number],
      default: 0
    },
    currentTimes: {
      type: [String, Number],
      default: 0
    },
    durationTime: {
      type: [String, Number],
      default: 0
    }
  },
  mounted() {
    this.$axios
      .get(this.HOST + "/v1/restserver/ting", {
        params: {
          method: "baidu.ting.song.lry",
          songid: this.id
        }
      })
      .then(res => {
        // 数据解析
        // 25:"哪会怕有一天只你共我"
        this.setLRC(res.data);
      })
      .catch(error => {
        console.log(error);
      });
  },
  methods: {
    setLRC(data) {
      var lyrics = data.lrcContent.split("\n");
      // 创建正则规则
      var lrcObj = {}; // 存储对象
      for (let i = 0; i < lyrics.length; i++) {
        var lyric = decodeURIComponent(lyrics[i]);
        var timeReg = /\[\d*:\d*((\.|\:)\d*)*\]/g; // [00:11.22]  [00:11:22]
        // 时间
        var timeRegExpArr = lyric.match(timeReg); // test
        if (!timeRegExpArr) continue; // break
        // 歌词
        var clause = lyric.replace(timeReg, "");
        // 时间字符串转化可对比的时间
        var t = timeRegExpArr[0];
        var min = Number(String(t.match(/\[\d*/i)).slice(1));
        var sec = Number(String(t.match(/\:\d*/i)).slice(1));
        var time = min * 60 + sec;
        lrcObj[time] = clause;
      }
      this.lrcData = lrcObj;
    }
  }
};
</script>
<style scoped>
.active {
  color: red;
}

.lrcContainer {
  width: 100%;
  height: 150px;
  overflow: hidden;
  position: relative;
}

.lrc {
  width: 100%;
  position: absolute;
  top: 0;
}

.lrc-p {
  height: 30px;
  line-height: 30px;
}

.up30 {
  margin-top: -30px;
}
</style>
