<template>
  <div>
    <audio
      src="http://mp3.9ku.com/hot/2010/05-27/315871.mp3"
      controls="controls"
      ref="audio"
      preload="auto"
      autoplay="autoplay"
      @timeupdate="timeupdate"
    ></audio>
    <!-- <button @click="play">play</button>
    <button @click="pause">pause</button> -->
    <br /><br />
    <div id="lrcArea">
      <!-- <div id="divide" /> -->
      <div id="lyric">
        <ul :style="styleObject">
          <li
            v-for="(lrc, i) in lrcs"
            :key="i"
            :class="{ actived: i === highlighted }"
            @mouseup="mouseUp"
            @mousedown="mouseDown($event, lrc[0])"
          >
            {{ lrc[1] }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script>
import lrc from "../lrc/index";
export default {
  name: "Music",
  data() {
    return {
      audio: null,
      lrcs: null,
      highlighted: null,
      downTime: null,
      upTime: null,
      downPosition: null,
      upPosition: null,
      currlrcTime: null,
      offset:0
    };
  },
  computed: {
    styleObject() {
      const { highlighted ,offset} = this;
      if (!highlighted || highlighted < 9) {
        return { transform: `translateY(0)` };
      }
      else if(offset&&(offset/24)>=6){
        return { transform: `translateY(0)` };
      }
      return {
        transform: `translateY(${0 - (highlighted - 9) * 24}px)`,
      };
    },
  },
  created() {
    this.$nextTick(() => {
      this.audio = this.$refs.audio;
      this.lrcs = this.parseLyric(lrc.shuangJieGun);
    });
  },
  watch: {
    upTime: function (val) {
      //是否长按
      if (val - this.downTime >= 300) {
        if (this.downPosition === this.upPosition) {
          this.audio.currentTime = this.currlrcTime;
        } else {
          const direction = this.downPosition - this.upPosition > 0 //true时向上，flase向下
          const offsetY = Math.abs(this.downPosition - this.upPosition);
          const lines = Math.floor(offsetY / 24);
          this.offset = offsetY
          // if(direction){
          //    this.highlighted -= lines
          // }else {
          //   this.highlighted += lines
          // }
         
        }
      }
      //点击事件（没有长按）
      else {
        this.audio.currentTime = this.currlrcTime;
      }
    },
  },
  methods: {
    play() {
      this.audio.play();
    },
    pause() {
      this.audio.pause();
    },
    mouseUp({ timeStamp, y }) {
      this.upTime = timeStamp;
      this.upPosition = y;
    },
    mouseDown({ timeStamp, y }, value) {
      this.downTime = timeStamp;
      this.currlrcTime = value;
      this.downPosition = y;
      // console.log(timeStamp, y, "down");
    },
    timeupdate({ target, offsetY }) {
      const { currentTime } = target;
      const { lrcs, audio } = this;
      if (lrcs && lrcs.length) {
        for (let i = 0; i < lrcs.length; i++) {
          const next = lrcs[i + 1] && lrcs[i + 1][0];
          const current = lrcs[i][0];
          const last = lrcs[lrcs.length - 1][0];
          if (
            (currentTime < next && currentTime > current) || //非最后一列
            (currentTime < audio.duration && currentTime > last) //最后一列
          ) {
            // console.log(lrcs[i][1])
            this.highlighted = i;
          }
        }
      }
    },
    parseLyric(text) {
      //将文本分隔成一行一行，存入数组
      var lines = text.split("\n"),
        //用于匹配时间的正则表达式，匹配的结果类似[xx:xx.xx]
        pattern = /\[\d{2}:\d{2}.\d{2}\]/g,
        //保存最终结果的数组
        result = [];
      //去掉不含时间的行
      while (!pattern.test(lines[0])) {
        lines = lines.slice(1);
      }
      //上面用'\n'生成生成数组时，结果中最后一个为空元素，这里将去掉
      lines[lines.length - 1].length === 0 && lines.pop();
      lines.forEach(function (v /*数组元素值*/) {
        //提取出时间[xx:xx.xx]
        var time = v.match(pattern),
          //提取歌词
          value = v.replace(pattern, "");
        //因为一行里面可能有多个时间，所以time有可能是[xx:xx.xx][xx:xx.xx][xx:xx.xx]的形式，需要进一步分隔
        time.forEach(function (v1) {
          //去掉时间里的中括号得到xx:xx.xx
          var t = v1.slice(1, -1).split(":");
          //将结果压入最终数组
          result.push([parseInt(t[0], 10) * 60 + parseFloat(t[1]), value]);
        });
      });
      //最后将结果数组中的元素按时间大小排序，以便保存之后正常显示歌词
      result.sort(function (a, b) {
        return a[0] - b[0];
      });
      console.log(result);
      return result;
    },
  },
};
</script>
<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
ul {
  padding: 0;
  li {
    margin: 0;
    width: 460px;
    list-style: none;
    line-height: 1.5;
    cursor: pointer;
    user-select: none;
  }
  li.actived {
    font-size: 1.5em;
    color: red;
    -webkit-mask-image: -webkit-gradient(
      linear,
      0 0,
      0 bottom,
      from(rgba(0, 0, 255, 0)),
      to(red)
    );
  }
}
#lrcArea {
  width: 460px;
  margin: 0 auto;
  text-align: center;
  // border: 1px solid #000;
  height: 500px;
  overflow: hidden;
  #divide {
    position: absolute;
    width: 100%;
    height: 0px;
    border-top: 1px dashed #000;
    top: calc(50% + 3px);
  }
  ul {
    transition: 0.1s ease-out 0s;
  }
  &::-webkit-scrollbar {
    // display: none;
  }
}
</style>