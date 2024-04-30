<template>
  <div style="display: flex; width: 1400px;">
    <div class="music_player_box">
      <audio ref="audio" :src="currentTrack.url" @loadedmetadata="onLoadedMetadata" @timeupdate="updateCurrentTime" @ended="handleTrackEnd"></audio>

      <!-- 播放器控制部分 -->
      <div class="controls">
        <button @click="prevTrack" style="margin: 0 10px; cursor: pointer;">上一首</button>
        <button @click="togglePlay" style="margin: 0 10px; cursor: pointer;">{{ isPlaying ? '暂停' : '播放' }}</button>
        <button @click="nextTrack" style="margin: 0 10px; cursor: pointer;">下一首</button>

        <!-- 播放模式选择 -->
        <select v-model="playMode">
          <option value="sequence">顺序播放</option>
          <option value="repeat-one">单曲循环</option>
          <option value="repeat-all">列表循环</option>
          <option value="shuffle">随机播放</option>
        </select>

        <!-- 音量调整 -->
        <input type="range" min="0" max="1" step="0.01" v-model="volume" @input="changeVolume"/>
      </div>

      <!-- 歌曲进度条 -->
      <div class="progress-bar">
        <input
          type="range"
          :max="duration"
          :value="currentTime"
          @input="seek"
          step="0.1"
        />
      </div>

      <!-- 显示当前歌曲时间 -->
      <div class="time-display">
        {{ formatTime(currentTime) }} / {{ formatTime(duration) }}
      </div>
    </div>
    <!-- 显示歌词 -->
    <div class="lyrics-container" ref="lyricsContainer">
      <div class="lyrics" :style="{ top: `- ${lyricOffset}px` }">
        <div v-for="(line, index) in currentLyric" :key="index" :class="{ current: isCurrentLine(index) }">
          {{ line.text }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {ref, computed} from 'vue'

interface playerListType {
  name: string;
  url: string;
  album: string;
  duration: number;
  id: number;
  lyric: string;
  playCount: number;
  rating: number;
}

const playerList = ref<playerListType[]>([
  {
    name: '红日',
    url: 'http://m701.music.126.net/20240429142331/7e9f79b8ca19fc6c75d2c2e7a930fab0/jdymusic/obj/wo3DlMOGwrbDjj7DisKw/28482311207/b026/640d/9f9f/f5a4714596772ea02cdbfe36ae0e4c16.mp3',
    album: '红日',
    duration: 287933,
    id: 32689581,
    lyric: '[00:00.000] 作词 : 李克勤\n' +
      '[00:00.500] 作曲 : 立川俊之\n' +
      '[00:01.000] 编曲 : 方树梁\n' +
      '[00:01.500] 编曲 : 方树梁\n' +
      '[00:02.000] Publisher : TV Asahi Ms (Polygram)\n' +
      '[00:02.500]\n' +
      '[00:02.775] AH.. AH... AH\n' +
      '[00:08.700]\n' +
      '[00:20.839] 命运就算颠沛流离\n' +
      '[00:22.803] 命运就算曲折离奇\n' +
      '[00:24.766] 命运就算恐吓着你\n' +
      '[00:26.777] 做人没趣味\n' +
      '[00:28.272] 别流泪 心酸\n' +
      '[00:30.232] 更不应舍弃\n' +
      '[00:32.251] 我愿能一生永远陪伴你\n' +
      '[00:36.806] 命运就算颠沛流离\n' +
      '[00:38.809] 命运就算曲折离奇\n' +
      '[00:40.799] 命运就算恐吓着你\n' +
      '[00:42.715] 做人没趣味\n' +
      '[00:44.235] 别流泪 心酸\n' +
      '[00:46.225] 更不应舍弃\n' +
      '[00:48.313] 我愿能一生永远陪伴你\n' +
      '[00:52.774] AH.. OH.. AH.. OH.. AH\n' +
      '[01:00.831] 一生之中兜兜转转\n' +
      '[01:02.736] 哪会看清楚\n' +
      '[01:03.972] 彷徨时我也试过\n' +
      '[01:05.686] 独坐一角 像是没协助\n' +
      '[01:08.792] 在某年 那幼小的我\n' +
      '[01:12.029] 跌倒过几多几多落泪\n' +
      '[01:14.261] 在雨夜滂沱\n' +
      '[01:16.761] 一生之中弯弯曲曲\n' +
      '[01:18.770] 我也要走过\n' +
      '[01:19.999] 从何时有你有你\n' +
      '[01:21.732] 伴我给我热烈地拍和\n' +
      '[01:24.239] 像红日之火 燃点真的我\n' +
      '[01:28.263] 结伴行 千山也定能踏过\n' +
      '[01:31.758] 让晚风 轻轻吹过\n' +
      '[01:35.981] 伴送着清幽花香\n' +
      '[01:37.750] 像是在祝福你我\n' +
      '[01:39.973] 让晚星 轻轻闪过\n' +
      '[01:43.968] 闪出你每个希冀如浪花\n' +
      '[01:47.478] 快要沾湿我 WOO...\n' +
      '[01:52.677] 命运就算颠沛流离\n' +
      '[01:54.647] 命运就算曲折离奇\n' +
      '[01:56.659] 命运就算恐吓着你\n' +
      '[01:58.722] 做人没趣味\n' +
      '[02:00.165] 别流泪 心酸\n' +
      '[02:02.208] 更不应舍弃\n' +
      '[02:04.171] 我愿能一生永远陪伴你\n' +
      '[02:08.743] AH HA AH HA AH.. HA..\n' +
      '[02:16.711] 一生之中兜兜转转\n' +
      '[02:18.700] 哪会看清楚\n' +
      '[02:19.972] 彷徨时我也试过\n' +
      '[02:21.701] 独坐一角 像是没协助\n' +
      '[02:24.750] 在某年 那幼小的我\n' +
      '[02:27.986] 跌倒过几多几多落泪\n' +
      '[02:30.208] 在雨夜滂沱\n' +
      '[02:32.687] 一生之中弯弯曲曲\n' +
      '[02:34.702] 我也要走过\n' +
      '[02:35.969] 从何时有你有你\n' +
      '[02:37.742] 伴我给我热烈地拍和\n' +
      '[02:40.214] 像红日之火 燃点真的我\n' +
      '[02:44.201] 结伴行 千山也定能踏过\n' +
      '[02:47.697] 让晚风 轻轻吹过\n' +
      '[02:51.920] 伴送着清幽花香\n' +
      '[02:53.664] 像是在祝福你我\n' +
      '[02:55.963] 让晚星 轻轻闪过\n' +
      '[02:59.926] 闪出你每个希冀如浪花\n' +
      '[03:03.458] 快要沾湿我 WOO...\n' +
      '[03:08.676] 命运就算颠沛流离\n' +
      '[03:10.667] 命运就算曲折离奇\n' +
      '[03:12.627] 命运就算恐吓着你\n' +
      '[03:14.678] 做人没趣味\n' +
      '[03:16.179] 别流泪 心酸\n' +
      '[03:18.187] 更不应舍弃\n' +
      '[03:20.214] 我愿能一生永远陪伴你\n' +
      '[03:24.728] 命运就算颠沛流离\n' +
      '[03:26.678] 命运就算曲折离奇\n' +
      '[03:28.698] 命运就算恐吓着你\n' +
      '[03:30.638] 做人没趣味\n' +
      '[03:32.216] 别流泪 心酸\n' +
      '[03:34.149] 更不应舍弃\n' +
      '[03:36.214] 我愿能一生永远陪伴你\n' +
      '[03:40.694] 命运就算颠沛流离\n' +
      '[03:42.646] 命运就算曲折离奇\n' +
      '[03:44.659] 命运就算恐吓着你\n' +
      '[03:46.626] 做人没趣味\n' +
      '[03:48.088] 别流泪 心酸\n' +
      '[03:50.086] 更不应舍弃\n' +
      '[03:52.104] 我愿能一生永远陪伴你\n' +
      '[03:56.590] 命运就算颠沛流离\n' +
      '[03:58.636] 命运就算曲折离奇\n' +
      '[04:00.657] 命运就算恐吓着你\n' +
      '[04:02.626] 做人没趣味\n' +
      '[04:04.149] 别流泪 心酸\n' +
      '[04:06.068] 更不应舍弃\n' +
      '[04:08.108] 我愿能一生永远陪伴你\n' +
      '[04:12.637] 命运就算颠沛流离\n' +
      '[04:14.568] 命运就算曲折离奇\n' +
      '[04:16.591] 命运就算恐吓着你\n' +
      '[04:18.606] 做人没趣味\n' +
      '[04:20.140] 别流泪 心酸\n' +
      '[04:22.104] 更不应舍弃\n' +
      '[04:24.098] 我愿能一生永远陪伴你\n' +
      '[04:28.664] AH HA AH.. HA..\n' +
      '[04:32.725] AH.. AH. HA.....',
    playCount: 100,
    rating: 5
  },
  {
    name: '夜曲',
    url: 'https://music.163.com/song/media/outer/url?id=1967121037.mp3',
    album: '夜曲',
    duration: 108904,
    id: 1967121037,
    lyric: '[00:00.00] 作词 : 方文山\n' +
      '[00:01.00] 作曲 : 周杰伦\n' +
      '[00:21.55]一群嗜血的蚂蚁\n' +
      '[00:23.28]被腐肉所吸引\n' +
      '[00:24.74]我面无表情\n' +
      '[00:26.11]看孤独的风景\n' +
      '[00:27.71]失去你\n' +
      '[00:28.45]爱恨开始分明\n' +
      '[00:30.23]失去你\n' +
      '[00:30.83]还有什么事好关心\n' +
      '[00:33.19]当鸽子不再象征和平\n' +
      '[00:35.03]我终于被提醒\n' +
      '[00:36.35]广场上喂食的是秃鹰\n' +
      '[00:38.66]我用漂亮的押韵\n' +
      '[00:40.00]形容被掠夺一空的爱情\n' +
      '[00:41.85]\n' +
      '[00:43.58]啊 乌云开始遮蔽\n' +
      '[00:45.37]夜色不干净\n' +
      '[00:46.42]公园里葬礼的回音\n' +
      '[00:48.15]在漫天飞行\n' +
      '[00:49.16]送你的白色玫瑰\n' +
      '[00:50.58]在纯黑的环境凋零\n' +
      '[00:52.10]乌鸦在树枝上\n' +
      '[00:53.49]诡异得很安静\n' +
      '[00:54.59]静静听我黑色的大衣\n' +
      '[00:57.10]想温暖你\n' +
      '[00:57.86]\n' +
      '[00:59.10]走过的走过的生命\n' +
      '[01:00.73]啊 四周弥漫雾气\n' +
      '[01:02.05]我在空旷的墓地\n' +
      '[01:03.24]老去后还爱你\n' +
      '[01:04.58]\n' +
      '[01:05.67]为你弹奏肖邦的夜曲\n' +
      '[01:08.72]纪念我死去的爱情\n' +
      '[01:11.45]跟夜风一样的声音\n' +
      '[01:14.28]心碎得很好听\n' +
      '[01:17.00]手在键盘敲很轻\n' +
      '[01:19.78]我给的思念很小心\n' +
      '[01:22.03]\n' +
      '[01:22.53]你埋葬的地方叫幽冥\n' +
      '[01:26.11]\n' +
      '[01:26.74]为你弹奏肖邦的夜曲\n' +
      '[01:30.32]\n' +
      '[01:30.86]纪念我死去的爱情\n' +
      '[01:33.10]\n' +
      '[01:33.61]而我为你隐姓埋名\n' +
      '[01:36.39]在月光下弹琴\n' +
      '[01:39.07]对你心跳的感应\n' +
      '[01:41.34]\n' +
      '[01:41.89]还是如此温热亲近\n' +
      '[01:44.58]怀念你那鲜红的唇印',
    playCount: 100,
    rating: 5
  },
  {
    name: '她说',
    url: 'http://m7.music.126.net/20240429142429/4d271659f7c332b4b6434e328c0ad9a3/ymusic/f704/0445/c5a9/6b5b0921c02285a6331e98b20537c9b5.mp3',
    album: '她说',
    duration: 178860,
    id: 33682191,
    lyric: '[00:00.490]她静悄悄地来过\n' +
      '[00:07.00]她慢慢带走沉默\n' +
      '[00:12.00]只是最后的承诺\n' +
      '[00:18.00]还是没有带走了寂寞\n' +
      '[00:25.00]我们爱的没有错\n' +
      '[00:30.30]只是美丽的独秀 太折磨\n' +
      '[00:37.00]她说无所谓\n' +
      '[00:42.00]只要能在夜里 翻来覆去的时候有寄托\n' +
      '[00:49.00]等不到天黑\n' +
      '[00:51.00]烟火不会太完美\n' +
      '[00:54.00]回忆烧成灰\n' +
      '[00:57.00]还是等不到结尾\n' +
      '[01:01.00]她曾说的无所谓\n' +
      '[01:04.500]我怕一天一天被摧毁\n' +
      '[01:12.300]等不到天黑\n' +
      '[01:15.00]不敢凋谢的花蕾\n' +
      '[01:18.600]绿叶在跟随 放开刺痛的滋味\n' +
      '[01:23.00]今后不再怕天明\n' +
      '[01:29.700]我想只是害怕清醒\n' +
      '[01:36.600]等不到天黑\n' +
      '[01:39.500]烟火不会太完美\n' +
      '[01:43.300]回忆烧成灰\n' +
      '[01:45.300]还是等不到结尾\n' +
      '[01:50.00]她曾说的无所谓\n' +
      '[01:53.00]我怕一天一天被摧毁\n' +
      '[02:01.00]等不到天黑 不敢凋谢的花蕾\n' +
      '[02:07.00]绿叶在跟随 放开刺痛的滋味\n' +
      '[02:12.00]今后不再怕天明\n' +
      '[02:19.00]我想只是害怕清醒\n' +
      '[02:26.00]等不到天明\n' +
      '[02:32.00]我想只是害怕  清醒\n' +
      '[02:40.00] wow ho',
    playCount: 100,
    rating: 5
  },
]);

const currentTrackIndex = ref(0);
const isPlaying = ref(false);
const volume = ref(0.5);
const playMode = ref('sequence');
const currentTime = ref(0);
const duration = ref(0); // 歌曲总时长

// 默认播放第一首
const currentTrack = computed(() => playerList.value[currentTrackIndex.value]);
// 监听音频元数据加载完成
const onLoadedMetadata = (event: Event) => {
  const audio = event.target as HTMLAudioElement;
  if (audio) {
    duration.value = audio.duration;
    if (isPlaying.value) {
      audio.play();
    }
  }
};

// 播放、暂停
const togglePlay = () => {
  const audio = document.querySelector('audio');
  if (isPlaying.value) {
    audio?.pause(); // 如果正在播放则暂停
  } else {
    audio?.play(); // 否则播放
  }
  isPlaying.value = !isPlaying.value;
};
// 上一首
const prevTrack = async () => {
  const audio = document.querySelector('audio');
  if (audio) {
    audio.pause(); // 切换前先暂停
  }
  currentTrackIndex.value = (currentTrackIndex.value - 1 + playerList.value.length) % playerList.value.length;
  await playCurrentTrack(); // 切换后自动播放
};
// 下一首
const nextTrack = async () => {
  const audio = document.querySelector('audio');
  if (audio) {
    audio.pause(); // 切换前先暂停
  }
  currentTrackIndex.value = (currentTrackIndex.value + 1) % playerList.value.length;
  await playCurrentTrack(); // 切换后自动播放
};

// 播放当前音乐
const playCurrentTrack = async () => {
  const audio = document.querySelector('audio');
  if (audio) {
    isPlaying.value = true;
    await audio.load(); // 确保加载完毕
    // 尝试播放
    await audio.play().catch((error) => {
      console.error("==============Error playing audio:", error.message);
      // 处理异常，例如重试、显示用户消息等
    });
  }
};
// 监听当前歌曲的播放进度
const updateCurrentTime = (event: Event) => {
  const audio = event.target as HTMLAudioElement;
  currentTime.value = audio.currentTime;
  updateLyricOffset(); // 根据播放进度更新歌词滚动
};
// 调节音量
const changeVolume = () => {
  const audio = document.querySelector('audio');
  if (audio) {
    audio.volume = volume.value;
  }
};
// 监听歌曲播放结束
const handleTrackEnd = () => {
  switch (playMode.value) {
    // 顺序播放
    case 'sequence':
      nextTrack();
      lyricOffset.value = 0;
      break;
    // 单曲循环
    case 'repeat-one':
      playCurrentTrack();
      lyricOffset.value = 0;
      break;
    // 列表循环
    case 'repeat-all':
      nextTrack();
      lyricOffset.value = 0;
      break;
    // 随机播放
    case 'shuffle':
      const randomIndex = Math.floor(Math.random() * playerList.value.length);
      currentTrackIndex.value = randomIndex;
      playCurrentTrack();
      lyricOffset.value = 0;
      break;
  }
};
// 调节播放进度
const seek = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const audio = document.querySelector('audio');
  if (audio) {
    audio.currentTime = parseFloat(target.value); // 跳转到新的播放位置
  }
};
// 格式化时间
const formatTime = (seconds: number) => {
  const mins = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60);
  return `${mins}:${secs.toString().padStart(2, '0')}`; // 格式化为分:秒
};

// 解析歌词
const currentLyric = computed(() => {
  const obj = currentTrack.value;
  return obj ? parseLyrics(obj.lyric) : [];
});

// 解析歌词并生成带有时间戳和内容的列表
const parseLyrics = (lyric: any) => {
  const lines = lyric.split('\n');
  const parsed = lines.map((line: any) => {
    const match = line.match(/\[(\d+):(\d+\.\d+)\] ?(.*)/); // 只需匹配文本部分
    if (match) {
      const [, mins, secs, text] = match;
      return {
        time: parseFloat(mins) * 60 + parseFloat(secs),
        text: text.trim(), // 确保去掉多余的空格
      };
    }
    return null; // 没有匹配成功的行返回 null
  }).filter((item: any) => item && item.text.length > 0); // 过滤掉空行或没有文本的行
  return parsed;
};

// 滚动偏移量
const lyricOffset = ref(0);

// 滚动歌词容器的引用
const lyricsContainer = ref<HTMLDivElement | null>(null);

// 更新歌词偏移，确保当前歌词在视野中
const updateLyricOffset = () => {
  const currentLyricIndex = currentLyric.value.findIndex((line: any) => line.time > currentTime.value);
  // 防止歌曲结束后偏移量重置
  if (currentTime.value <= duration.value) {
    if (currentLyricIndex > 0) {
      lyricOffset.value = (currentLyricIndex - 1) * 30; // 30px 代表每行歌词的高度
    }
  } else {
    // 如果歌曲结束，保持在最后一行
    lyricOffset.value = (currentLyric.value.length - 1) * 30;
  }
};

// 判断当前行是否为当前时间对应的歌词行
const isCurrentLine = (index: number) => {
  const nextIndex = currentLyric.value.findIndex((line: any) => line.time > currentTime.value);
  if (currentTime.value <= duration.value) {
    if (nextIndex > 0) {
      return index === nextIndex - 1; // 当前时间对应的歌词行
    } else {
      if (currentTime.value > 0) {
        return index === currentLyric.value.length - 1;
      }
    }
  }
};

</script>

<style scoped>
.music_player_box {
  background-color: white;
}

.controls {
  display: flex;
  gap: 10px;
}

.progress-bar {
  margin-top: 10px;
}


.lyrics-container {
  flex: 1;
  height: 500px;
  overflow-y: auto;
  position: relative;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.lyrics-container:active {
  cursor: grab;
}

.lyrics {
  position: absolute;
  transition: top 0.3s linear;
}

.lyrics > div {
  padding: 5px;
  text-align: center;
}

.current {
  color: red;
}

.time-display {
  margin-top: 5px;
}
</style>
