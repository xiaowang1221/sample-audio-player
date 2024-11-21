<script lang="ts" setup>
import {ref, onMounted} from 'vue';
import {ElNotification} from 'element-plus';


// 定义 DOM 引用和文件列表
const audioFileInput = ref<HTMLInputElement | null>(null);
const audioPlayer = ref<HTMLAudioElement | null>(null);
const playlist = ref<HTMLUListElement | null>(null);
const audioName = ref<HTMLHeadingElement | null>(null);
const files = ref<File[]>([]);
const currentIndex = ref<number | null>(null); // 当前播放的音乐索引


// 文件改变事件
function handleFileChange(event: Event) {
  const input = event.target as HTMLInputElement;
  const newFiles = Array.from(input.files || []);
  files.value = files.value.concat(newFiles);
  console.log(files.value)
  updatePlaylist();
  openSuccess();
}


// 更新播放列表
function updatePlaylist() {
  if (!playlist.value) return;
  playlist.value.innerHTML = ''; // 清空播放列表
  files.value.forEach((file, index) => {
    const listItem = document.createElement('li');
    listItem.classList.add('playlist-item');
    // 创建文件名部分
    const text = document.createElement('span');
    text.textContent = file.name;
    listItem.appendChild(text);
    // 播放音频
    listItem.addEventListener('click', () => playAudio(index));

    playlist.value?.appendChild(listItem);
  });
}



// 删除音乐
function deleteAudio(index: number) {
  if (index === currentIndex.value) {
    // 如果删除的是当前播放的音乐，播放下一首
    playNextAudio()
  }

  // 从文件列表中移除该项
  files.value.splice(index, 1);
  delSuccess()

  // 如果列表还有歌曲，且当前播放索引未定义，自动设置下一首
  if (currentIndex.value !== null && currentIndex.value >= index) {
    currentIndex.value = currentIndex.value > 0 ? currentIndex.value - 1 : null;
  }

  // 更新列表并重新设置播放状态样式
  updatePlaylist();

  // 如果仍有播放索引，重新应用样式
  if (currentIndex.value !== null) {
    const items = playlist.value?.getElementsByClassName('playlist-item') || [];
    items[currentIndex.value]?.classList.add('playing');
  }
}


// 播放音频
function playAudio(index: number) {
  const file = files.value[index];
  if (!audioPlayer.value || !audioName.value) return;
  // 设置音频源并播放
  audioPlayer.value.src = URL.createObjectURL(file);
  audioPlayer.value.play();
  currentIndex.value = index; // 更新当前播放索引
  audioName.value.textContent = file.name;
  // 更新播放列表样式
  const items = playlist.value?.getElementsByClassName('playlist-item') || [];
  Array.from(items).forEach(item => item.classList.remove('playing'));
  items[index]?.classList.add('playing');
}


// 清除播放列表
function clearPlaylist() {
  files.value = [];
  currentIndex.value = null;
  updatePlaylist();
  if (audioPlayer.value) {
    audioPlayer.value.pause();
    audioPlayer.value.src = '';
  }
  if (audioName.value) {
    audioName.value.textContent = '选择音乐';
  }
}


// 自动播放下一个音频
function playNextAudio() {
  if (currentIndex.value === null || !audioPlayer.value) return;
  const nextIndex = currentIndex.value + 1;
  if (nextIndex < files.value.length) {
    playAudio(nextIndex);
  } else {
    // 如果已经是最后一个音频，则停止播放
    currentIndex.value = null;
    audioPlayer.value.src = '';
    if (audioName.value) {
      audioName.value.textContent = '播放结束';
    }
  }
}


// 设置生命周期，在组件挂载时绑定事件
onMounted(() => {
  if (audioPlayer.value) {
    audioPlayer.value.addEventListener('ended', playNextAudio);
  }
});


// 显示添加成功通知
const openSuccess = () => {
  ElNotification({
    title: '提示',
    message: '已添加歌曲',
    type: 'success',
  });
};


// 显示删除通知
const delSuccess = () => {
  ElNotification({
    title: '删除',
    message: '已删除歌曲',
    type: 'warning',
  });
};

</script>

<template>
  <div class="common-layout">

    <el-container>
      <el-header>
        <!-- 音乐名 -->
        <div class="music-name">
          <h2 id="audio-name" ref="audioName">选择音乐</h2>
        </div>
      </el-header>

      <el-main>
        <!-- 选择文件 -->
        <div class="select-audio">
          <el-button class="custom-file-button" plain type="primary" @click="audioFileInput?.click()">添加文件
          </el-button>
          <el-button class="clear-button" plain type="danger" @click="clearPlaylist">清除列表</el-button>
          <input id="audioFile" ref="audioFileInput" accept="audio/*" multiple type="file" @change="handleFileChange"/>
        </div>

        <!-- 播放列表 -->
        <div class="playlist-container">
          <el-scrollbar height="300px">
          <ul class="playlist">
            <li
              v-for="(file, index) in files"
              :key="index"
              :class="{ 'playlist-item': true, playing: index === currentIndex }"
              @click="playAudio(index)"
            >
            <span>{{ file.name }}</span>
            <el-button
              type="danger"
              @click.stop="deleteAudio(index)"
              plain
            >删除</el-button>
            </li>
          </ul>
          </el-scrollbar>
        </div>
      </el-main>

      <el-footer>
        <!-- 播放栏 -->
        <div class="audio-bar">
          <audio id="audioPlayer" ref="audioPlayer" controls></audio>
        </div>
      </el-footer>
    </el-container>
  </div>

</template>


<style>
/**
  歌名部分
*/
.music-name {
  display: flex;
  justify-content: center;
  /* 水平居中 */
  align-items: center;
  /* 垂直居中 */
  height: 100%;
  /* 确保容器有高度 */
}

#audio-name {
  margin: 0;
  /* 清除默认的 margin */
}

/* 自定义选择文件按钮样式 */
#audioFile {
  display: none;
}


/**
  资源管理按钮
*/
.select-audio {
  margin-bottom: 10px;
}



/**
  播放栏
*/
.audio-bar {
  width: 100%;
  /* 让容器宽度充满父级元素 */
  max-width: 600px;
  /* 限制最大宽度 */
  margin: 0 auto;
  /* 居中容器 */
  padding: 10px;
  /* 内边距 */
  border-radius: 8px;
  /* 圆角 */
}

#audioPlayer {
  width: 100%;
  /* 让音频控件占满容器宽度 */
  height: 40px;
  /* 设置播放器高度 */
  border-radius: 5px;
  /* 圆角 */
  background-color: #fff;
  /* 背景色 */
  border: none;
  /* 移除默认边框 */
}

/**
  播放列表部分
*/
/* 隐藏原生滚动条 */
.playlist {
  /*  overflow: hidden; */
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px 0;
  background-color: #f9f9f9;
}

.playlist-item {
  display: flex;
  /* 使用 Flexbox 布局 */
  justify-content: space-between;
  /* 左右分布 */
  align-items: center;
  /* 垂直居中 */
  cursor: pointer;
  padding: 5px;
  border-bottom: 1px solid #eee;
}

.playlist-item:hover {
  background-color: #e6f7ff;
}

.playlist-item.playing {
  font-weight: bold;
  color: #409EFF;
}

</style>