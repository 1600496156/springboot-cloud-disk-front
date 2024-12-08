<template>
  <span class="avatar" :style="{ width: width + 'px', height: width + 'px' }">
    <img
        v-if="avatarData && avatarData.data"
        :src="avatarData.data"
        :alt="'User avatar for ' + userId"
    />
  </span>
</template>

<script setup>
import { getCurrentInstance } from "vue";
const { proxy } = getCurrentInstance();
import { ref, onMounted } from 'vue';
import axios from 'axios';

const props = defineProps({
  userId: {
    type: String,
    required: true,
  },
  avatar: {
    type: String,
  },
  timestamp: {
    type: Number,
    default: 0,
  },
  width: {
    type: Number,
    default: 40,
  },
});

const avatarData = ref(null);

// 发送请求获取头像数据的函数
async function fetchAvatarData() {
  try {
    const response = await axios.get(`/api/avatar?userId=${props.userId}`); // 假设的API端点
    if (response.data && response.data.code === 200) {
      avatarData.value = response.data;
    } else {
      console.error('Error fetching avatar data:', response.data);
      // 处理错误情况，比如设置一个默认头像或显示错误信息
    }
  } catch (error) {
    console.error('Error fetching avatar data:', error);
    // 处理请求错误，比如设置一个默认头像或显示错误信息
  }
}

// 在组件挂载后获取头像数据
onMounted(fetchAvatarData);

</script>

<style lang="scss" scoped>
.avatar {
  display: flex;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  overflow: hidden;
  img {
    width: 100%;
    object-fit: cover;
  }
}
</style>