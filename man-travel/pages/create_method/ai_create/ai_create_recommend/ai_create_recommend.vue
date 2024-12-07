<template>
  <view class="container">
    <!-- 关闭按钮 -->
    <view class="close-icon-container">
      <image src="/static/icons/close.png" class="close-icon" @click="closePage" />
    </view>

    <!-- 标题：这趟旅行要去哪里？ -->
    <view class="title-container">
      <text class="title-text">这趟旅行要去哪里？</text>
    </view>

    <!-- 旅行目的地 -->
    <view class="location-container">
      <view class="location-circle">
        <text class="location-text">{{ locationInput }}</text>
      </view>
    </view>

    <!-- 标题：你希望这趟旅行？ -->
    <view class="travel-preference-title-container">
      <text class="travel-preference-title">你希望这趟旅行？</text>
    </view>

    <!-- 旅行关键词块 -->
    <view class="keywords-container">
      <view class="keyword-item" 
            v-for="(keyword, index) in travelKeywords.slice(0, 9)" 
            :key="index" 
            :class="{'selected': selectedKeywords.includes(keyword)}" 
            @click="selectKeyword(keyword)">
        <text class="keyword-text">{{ keyword }}</text>
      </view>
    </view>

    <!-- 标题：这些地点不容错过！ -->
    <view class="must-visit-title-container">
      <text class="must-visit-title">这些地点不容错过！</text>
    </view>

    <!-- 热门景点列表 -->
    <scroll-view class="locations-container" scroll-y="true">
      <view class="location-item" v-for="(location, index) in popularLocations" :key="index">
        <view class="location-info">
          <text class="location-name">{{ location.name }}</text>
          <view class="location-details">
            <text class="location-type" :style="getLocationTypeColor(location.type)">{{ location.type }}</text>
            <text class="location-separator"> |  </text>
            <text class="location-recommendation">{{  location.recommendation }}</text>
          </view>
        </view>
        <!-- 选择框 -->
        <view class="select-icon-container" @click="toggleLocationSelection(index)">
          <image :src="location.isSelected ? '/static/icons/selected.png' : '/static/icons/select.png'" class="select-icon" />
        </view>
      </view>
    </scroll-view>

    <!-- 开始规划按钮 -->
    <view class="start-planning-container">
      <view class="start-planning-button" @click="startPlanning">
        <!-- 魔法棒图标 -->
        <image src="/static/icons/magic-wand-green.png" class="magic-wand-icon" />
        <text class="start-planning-text">开始规划！</text>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { onLoad } from '@dcloudio/uni-app';

const locationInput = ref('');
const startDate = ref('');
const endDate = ref('');
const dayCount = ref(0);

onLoad((options) => {
  if (options.data) {
    // 解码并解析接收到的数据
    const decodedData = decodeURIComponent(options.data);
    const travelData = JSON.parse(decodedData);

    // 将数据赋值给相应的变量
    locationInput.value = travelData.city || '';

    // 解析并格式化日期为 'YYYY-MM-DD'
    if (travelData.startDate) {
      const startDateObj = new Date(travelData.startDate);
      startDate.value = startDateObj.toISOString().split('T')[0]; // 提取日期部分
    } else {
      startDate.value = '';
    }

    if (travelData.endDate) {
      const endDateObj = new Date(travelData.endDate);
      endDate.value = endDateObj.toISOString().split('T')[0]; // 提取日期部分
    } else {
      endDate.value = '';
    }

    dayCount.value = travelData.dayCount || 0;

    // 打印接收到的旅行数据
    console.log('接收到的旅行数据:', travelData);
  } else {
    console.error('未接收到旅行数据');
    uni.showToast({
      title: '未收到旅行数据，请返回重新选择',
      icon: 'none',
    });
  }
});

// 旅行关键词词库
const keywordPool = [
  '乡村风情', '呼吸自然', '美食之旅', '小众地点', '海岛风光', '特种兵打卡',   
  '慢时光', '名胜古迹', 'CityWalk', '自然风光', '自然风光', '主题公园', 
  '动物园探访', '博物馆之旅', '购物狂欢', '奢华度假', '温馨亲子', '生态观光', 
  '田园牧歌', '宗教朝圣', '夜市美食', '夜空观星', '乡野采摘', '地道手工', 
  '自驾之旅', '动物观察', '山川峡谷', '音乐节日', '建筑之美', '摄影打卡', 
  '露营体验', '手工艺品探店', '寺庙参观', '经典建筑',
];

// 存储随机选择的9个关键词
const travelKeywords = ref([]);
const selectedKeywords = ref([]);

// 初始化随机关键词
const initializeKeywords = () => {
  travelKeywords.value = keywordPool.sort(() => Math.random() - 0.5).slice(0, 9);
};

// 存储热门景点数据
const popularLocations = ref([]);

onMounted(() => {
  // 从缓存中获取用户选择的城市
  locationInput.value = uni.getStorageSync('selectedCity') || '未选择城市'; // 从缓存中获取城市
  initializeKeywords(); // 初始化关键词

  // 从缓存中获取热门景点数据
  const sceneData = uni.getStorageSync('popularLocations');
  console.log('从缓存中获取的数据:', sceneData); // 打印缓存的数据

  if (sceneData) {
    try {
      // 解析缓存数据
      const parsedData = JSON.parse(sceneData); // 解析缓存数据

      if (parsedData && Array.isArray(parsedData.positions)) {
        // 给每个景点数据添加缺失字段，默认设置 isSelected 为 false
        popularLocations.value = parsedData.positions.map(location => ({
          name: location.name,
          type: location.tag, // 如果缓存中的字段是 tag，需要映射到 type
          recommendation: location.description, // 如果缓存中的字段是 description，需要映射到 recommendation
          isSelected: location.isSelected || false, // 默认值 false
        }));

        console.log('解析后的热门景点数据:', popularLocations.value); // 打印解析后的数据
      } else {
        throw new Error('数据格式不正确');
      }
    } catch (error) {
      console.error('解析缓存数据失败:', error);
      uni.showToast({
        title: '数据解析失败，请重试',
        icon: 'none'
      });
    }
  } else {
    console.log('未找到缓存数据');
    uni.showToast({
      title: '未找到热门景点数据',
      icon: 'none'
    });
  }
});

// 切换景点选择状态
const toggleLocationSelection = (index) => {
  popularLocations.value[index].isSelected = !popularLocations.value[index].isSelected;
};

// 获取地点类型颜色
const getLocationTypeColor = (type) => {
  if (type === '景点') {
    return 'color: #54BCBD;';
  } else if (type === '吃喝') {
    return 'color: #E99D42;';
  }
  return 'color: #000;';
};

// 选择旅行关键词
const selectKeyword = (keyword) => {
  if (selectedKeywords.value.includes(keyword)) {
    // 如果已经选中，则取消选中
    selectedKeywords.value = selectedKeywords.value.filter(item => item !== keyword);
  } else {
    // 如果未选中，则添加到选中列表
    selectedKeywords.value.push(keyword);
  }
};

const isLoading = ref(false); // 添加加载状态

const startPlanning = () => {
  // 检查是否正在加载，避免重复提交
  if (isLoading.value) {
    return;
  }

  // 获取本地存储中的 access_token
  const token = uni.getStorageSync('access_token');
  if (!token) {
    uni.showToast({
      title: '请先登录',
      icon: 'none',
    });
    return;
  }

  // 获取已选择的关键词，并用逗号隔开
  const selectedKeywordsList = selectedKeywords.value.join('，');

  // 获取已选择的景点名称，并用逗号隔开
  const selectedLocationNames = popularLocations.value
    .filter(location => location.isSelected)
    .map(location => location.name)
    .join('，');

  // 使用接收到的开始和结束日期
  const travelDates = { startDate: startDate.value, endDate: endDate.value };

  // 生成用户行程数据
  const userItinerary = {
    location: locationInput.value,
    travelDates: travelDates,
    dayCount: dayCount.value,
    keywords: selectedKeywordsList, // 字符串形式的关键词
    selectedLocations: selectedLocationNames, // 字符串形式的景点名称
  };

  // 打印完整的用户行程数据
  console.log('用户行程:', JSON.stringify(userItinerary, null, 2));

  // 禁用按钮，避免多次点击
  isLoading.value = true;

  // 跳转至 loading 页面
  uni.navigateTo({
    url: '/pages/loading/loading', // 跳转到 loading 页面
  });

  // 准备上传的数据
  const dataToUpload = {
    trip_name: userItinerary.location,                // 地点
    start_date: userItinerary.travelDates.startDate,  // 开始日期
    end_date: userItinerary.travelDates.endDate,      // 结束日期
    day_counts: userItinerary.dayCount,               // 天数
    style: userItinerary.keywords,                    // 关键词字符串
    attractions: userItinerary.selectedLocations,     // 景点名称字符串
  };

  // 打印准备上传的数据
  console.log('准备上传的数据:', JSON.stringify(dataToUpload, null, 2));

  // 发送 POST 请求，将行程数据上传到服务器
  uni.request({
    url: 'https://734dw56037em.vicp.fun/api/trip/AiCreateStyles/', // 替换为实际的后端接口 URL
    method: 'POST',
    data: dataToUpload,
    header: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
    },
    // 在成功回调中获取 trip_id 并导航
        success: (res) => {
          console.log('服务器响应:', res);
          if (res.statusCode === 201) {
            // 假设服务器在响应体中返回了 trip_id
            const tripId = res.data.trip_id;
			console.log('trip_id=',tripId);
            uni.showToast({
              title: '行程创建成功',
              icon: 'success',
            });
    
            // 跳转到 Overview 页面，并传递 trip_id 作为参数
            uni.navigateTo({
              url: `/pages/Overview/Overview?trip_id=${tripId}` // 传递 trip_id
            });
          } else {
            // 打印详细错误信息
            console.error('创建行程失败:', res.data);
            // 显示服务器返回的错误信息
            uni.showToast({
              title: res.data.error || res.data.detail || '行程创建失败，请稍后再试',
              icon: 'none',
            });
          }
        },
        fail: (err) => {
          // 请求失败处理
          console.error('请求失败:', err);
          uni.showToast({
            title: '网络请求失败，请检查网络连接',
            icon: 'none',
          });
        },
        complete: () => {
          // 请求完成后，重置加载状态
          isLoading.value = false;
        },
      });
    };
</script>



<style scoped>
/* 页面整体布局 */
.container {
  background-color: rgba(180, 253, 255, 0.20); /* 设置背景色的透明度为15% */
  padding: 40rpx;
  display: flex;
  flex-direction: column;
  justify-content: flex-start; /* 保持顶部内容固定 */
  height: 100vh;
  position: relative;
  overflow: hidden;
}

/* 关闭按钮 */
.close-icon-container {
  position: absolute;
  top: 20rpx;
  left: 20rpx;
  z-index: 100;
}

.close-icon {
  width: 40rpx;
  height: 40rpx;
  cursor: pointer;
}

/* 标题：这趟旅行要去哪里？ */
.title-container {
  margin-top: 80rpx; /* 调整顶部间距 */
  display: flex;
  font-family: 'TaipeiSansTCBeta';
}

.title-text {
  font-size: 54rpx;
  font-weight: bold;
  color: #000;
}

/* 旅行目的地显示 */
.location-container {
  margin-top: 30rpx;
  display: flex;
  margin-left: 20rpx;
}
.location-circle {
  display: inline-flex; /* 使容器宽度自适应内容 */
  align-items: center;
  justify-content: center;
  padding: 0 20rpx; /* 给内容左右增加额外的空间，使宽度稍微大于内容 */
  height: 75rpx;
  border-radius: 35rpx;
  border: 1px solid rgba(128, 128, 128, 0.5);
  background-color: white;
}

.location-text {
  font-size: 36rpx;
  color: #000;
  font-weight: 550;
}

/* 标题：你希望这趟旅行？ */
.travel-preference-title-container {
  margin-top: 40rpx;
  display: flex;
  justify-content: flex-start; /* 左对齐 */
  font-family: 'TaipeiSansTCBeta';
}

.travel-preference-title {
  font-size: 54rpx;
  font-weight: bold;
  color: #000;
}

/* 旅行关键词块 */
.keywords-container {
  margin-top: 40rpx;
  display: flex;
  flex-wrap: wrap;
  gap: 20rpx;
  margin-left: 20rpx;
}

.keyword-item {
  display: inline-flex; /* 使容器宽度自适应内容 */
  align-items: center;
  justify-content: center;
  padding: 0 20rpx; /* 给内容左右增加额外的空间，使宽度稍微大于内容 */
  height: 75rpx;
  border-radius: 35rpx;
  border: 1px solid rgba(128, 128, 128, 0.5);
  background-color: #fff;
  cursor: pointer;
  transition: all 0.3s ease;
}

.keyword-item.selected {
  background-color: #000;
  color: #fff;
  font-weight: bold;
}

.keyword-text {
  font-size: 32rpx;
  color: #000;
  font-weight: 500;
}

.keyword-item.selected .keyword-text {
  color: #fff;
}

/* 开始规划按钮 */
.start-planning-container {
  position: fixed;
  bottom: 55rpx;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: center;
}

.start-planning-button {
  height: 105rpx;
  border-radius: 30rpx;
  border: 2px solid #05AD8E;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 5rpx 7rpx rgba(0, 0, 0, 0.25); /* 添加阴影效果 */
}

.magic-wand-icon {
  margin-left: 22rpx;
  width: 75rpx;
  height: 75rpx;
  margin-right: 10rpx; /* 给魔法棒图标和文本之间添加间距 */
}

.start-planning-text {
  color: #05AD8E;
  font-size: 50rpx;
  font-weight: bold;
  letter-spacing: 1rpx; /* 设定字间距 */
}

/* 加载自定义字体 */
@font-face {
  font-family: 'TaipeiSansTCBeta';
  src: url('font/TaipeiSansTCBeta-Regular.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}

body {
  font-family: sans-serif; /* 默认字体 */
}

/* 标题：这些地点不容错过！ */
.must-visit-title-container {
  margin-top: 40rpx;
  display: flex;
  justify-content: flex-start;
  font-family: 'TaipeiSansTCBeta';
}

.must-visit-title {
  font-size: 54rpx;
  font-weight: bold;
  color: #000;
}

/* 热门景点列表 */
.locations-container {
  margin-top: 30rpx;
  max-height: 550rpx;
  overflow-y: scroll;
  padding-right: 20rpx;
}

.location-item {
  display: flex;
  align-items: center;
  padding: 20rpx;
  border-radius: 20rpx;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.01);
}

.location-thumbnail {
  width: 120rpx;
  height: 120rpx;
  border-radius: 20rpx;
  overflow: hidden;
  margin-right: 20rpx;
}

.thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.location-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.location-name {
  font-size: 36rpx;
  color: #000;
  margin-bottom: 10rpx;
}

.location-details {
  display: flex;
  align-items: center;
}

.location-type {
  font-size: 28rpx;
  font-weight: bold;
  margin-right: 10rpx;
}

.location-recommendation {
  font-size: 28rpx;
  color: #666;
  margin-left: 8rpx;
}

.select-icon-container {
  width: 60rpx;
  height: 60rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.select-icon {
  width: 100%;
  height: 100%;
}
</style>
