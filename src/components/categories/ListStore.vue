<template>
  <div class="w-2/3 mx-auto grid grid-cols-2 mb-3">
    <router-link v-for="store in filteredStores" :key="store.id" :to="getStoreDetailsLink(store.id)"
  :class="{'p-4 border border-gray-300 rounded-md flex items-center mx-2 my-1':store.status === 'OPEN',
  'p-4 border border-gray-300 bg-gray-100 rounded-md flex items-center mx-2 my-1':store.status === 'CLOSED',
  }">
  <img :src="getImage(store.id)" :class="{'w-[70px] h-[70px] bg-gray-100 object-cover mb-2 rounded-md mr-4': store.status === 'OPEN',
  'w-[70px] h-[70px] bg-gray-100 object-cover mb-2 rounded-md mr-4 brightness-50': store.status === 'CLOSED', }
  ">
  <div>
    <p class="text-gray-500">{{ store.name }}</p>
    <span class="font-bold text-yellow-500">★</span> 
    <span v-if="store.avgRating != 'NaN'" class="font-bold text-yellow-500">{{ store.avgRating }}</span>
    <span v-else class="font-bold text-yellow-500">0</span>
    <span class="font-bold text-red-500">&nbsp;♥</span> 
    <span class="font-bold text-red-500">{{ store.likes }}</span>    
    <p class="text-gray-500" style="font-size: 0.8em;">주문 수 : {{store.countOrder}}</p>
    <p v-if="store.status === 'CLOSED'" class="text-red-500">영업이 종료되었습니다.</p>
  </div>
</router-link>
  </div>
</template>

<script setup>
import { defineProps, ref, onMounted, watchEffect } from "vue";
import axios from 'axios';

const props = defineProps({
  category: String,
  searchName: String
});

const pageSize = 10; //페이지 사이즈 10 설정
let currentPage = 0 ;// 초기 페이지 0페이지 설정
let isLastPage = false;
let isLoading = false;
let isInitialLoad = true; // 초기 반복하는 현상 제거
const stores = ref([]); // 상점 데이터를 담을 ref
const filteredStores = ref([]); // 필터링된 상점 데이터를 담을 ref

onMounted(async () => { 
  if (isInitialLoad) {
    isInitialLoad = false;
  } else {
    fetchData();  // 초기 로드를 방지하기 위해 처음 렌더링 시 fetchData 호출 건너뛰기
  }
  window.addEventListener('scroll', scrollPagination);
});

// props의 변경을 감지하고 filteredStores를 업데이트
// 검색, 다른카테고리로 이동 했을 때 props가 변경하는 것 이기 때문에
// 페이지 관련 변수들을 초기화한다.
watchEffect(() => {
    stores.value = [];
    currentPage = 0;
    isLastPage = false;
    fetchData();
});
// 매장 상세 조회를 위한 함수
// 매장목록에 storeId를 받아 path에 해당 주소로 이동하고
// params로 해당 storeId를 id 변수로 선언한뒤 전달
const getStoreDetailsLink = (storeId) => {
  return { path: `/${storeId}/store`, params: { id: storeId }};
};
// 무한 스크롤 기능 구현 
const scrollPagination = async () => {
  const nearBottom = window.innerHeight + window.scrollY >= document.body.offsetHeight - 500;
  if (nearBottom && !isLastPage && !isLoading){
    currentPage ++;
    await fetchData(); // 하단에 도달하면 더 많은 데이터를로드합니다.
  }
};
// 이미지 조회 함수
const getImage = (id) => {
  return `${process.env.VUE_APP_API_BASE_URL}/api/stores/${id}/image`
};

// 목록조회 함수
async function fetchData() {
  isLoading = true;
  try {
    const params = { 
      name: props.searchName,
      category: props.category,
      page : currentPage,
      size : pageSize
    };
    const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/api/stores/`, { params });
    const addStoreList = response.data.result;
    if (addStoreList.length < pageSize) {
      isLastPage = true;
    }
    console.log("currentPage" + currentPage);
    stores.value = [...stores.value, ...addStoreList];
    filteredStores.value = stores.value;
  } catch (error) {
    console.error('데이터 조회 중 에러 발생:', error);
  }
  isLoading = false;
}
</script>