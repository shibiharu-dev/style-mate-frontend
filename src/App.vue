
<script setup>
// 服を表示するコンポーネントをインポートする
// ./は同じフォルダ内という意味
import ClothesList from './components/ClothesList.vue';
//状態を記憶するためにrefという特別な関数を使う
import { ref } from 'vue';

//選択中のIDを記憶する箱　初期値はnull
const selectedTopsId = ref(null);
const selectedBottomsId = ref(null);

//トップスがクリックされたときに動く関数
const handleTopsSelect = (id) => {         //関数の宣言　idは引数
  selectedTopsId.value = id;               //selectedTopIdのvalueにidをいれる
  selectedBottomsId.value = null;          //Topsが選択されたらBottomsの選択を外す
  console.log("選ばれたトップスのID:", id); //デバック用
};

//ボトムスがクリックされたときに動く関数
const handleBottomsSelect = (id) => {
  selectedBottomsId.value = id;
  selectedTopsId.value = null;
  console.warnlog("選ばれたボトムスのID:", id);
};

//提案するボタンがクリックされたときに動く関数
const makeProposal = () => {
  alert(`ID:${selectedTopsId.value} と ID:${selectedBottomsId.value} でコーディネートしました！`);
};

//トップス用のデータ
const topsData = [
  { id: 1, name: "白シャツ", color: "#ffffff"},
  { id: 2, name: "黒パーカー", color: "#333333"},
  { id: 3, name: "青シャツ", color: "#aaccff"},
  { id: 4, name: "赤ニット", color: "#ffcccc"},
];

//ボトムス用のデータ
const bottomsData = [
  { id: 1, name: "デニム", color: "#336699"},
  { id: 2, name: "カーゴパンツ", color: "#ddccaa"},
  { id: 3, name: "スカート", color: "ffaaaa"},
  { id: 4, name: "スカート", color: "ffaaaa"},
  { id: 4, name: "スカート", color: "ffaaaa"},
];
</script>

<template>
  <!-- ここには見た目（HTML）を書きます -->
  <div class="app-container">
    <!--ヘッダー部分-->
    <header class="site-header">
      <h1 class="site-title">StyleMate</h1>
    </header>


    <!--メインコンテンツ-->
    <main class= "main-content">
      <!--トップスエリア-->
      <div class="clothing-section">
        <h2>トップス</h2>
        <!-- :items="topsData" でデータを渡す！ -->
        <!-- :selected-id="selectedTopsId"で親が保存しているIDを渡す-->
        <!-- @select="handleTopsSelect" 選ばれたときにhandleTopsSelectを実行-->
        <ClothesList :items="topsData" :selected-id="selectedTopsId" @select="handleTopsSelect"/>
      </div>

      <!--ボトムスエリア-->
      <div class="clothing-section">
        <h2>ボトムス</h2>
        <!-- :items="bottomsData" でデータを渡す！ -->
        <ClothesList :items="bottomsData" :selected-id="selectedBottomsId" @select="handleBottomsSelect"/>
      </div>
      
      <!-- 提案ボタン-->
      <div class="action-area" v-if="selectedTopsId || selectedBottomsId">
        <button class="propose-btn" @click="makeProposal">
          コーディネートを提案する！
        </button>
      </div>


    </main>
  </div>
</template>

<style scoped>
/*ヘッダーのデザイン*/
.site-header{
  background-color: #0e0d0d;
  color: white;
  padding: 1rem;
  text-align: center;
}
/*メインエリア全体*/
.main-content {
  padding: 20px;
  max-width: 600px;
  margin: 0 auto;
}
/*トップス、ボトムスの各エリア*/
.clothing-section {
  margin-bottom: 30px;
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0,1) 
}
/*h2*/
h2 {/*注意ここには.をつけない*/
  margin-bottom: 10px;
  font-size: 1.2rem;
  border-bottom: 2px solid #ddd; /* 下線 */
}
/*仮デザイン
.placeholder-box {
  background-color: #eee;
  height: 300px;          
  display: flex;          
  align-items: center;   
  justify-content: center;
  color: #888;
}*/


/* アクションエリア */
.action-area {
  text-align: center;
  margin-top: 40px;
}

/* 提案ボタンのデザイン */
.propose-btn {
  background-color: #42b883;
  color: white;
  border: none;
  padding: 15px 30px;
  font-size: 1.2rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: bold;
  box-shadow: 0 4px 6px rgba(0,0,0,0.2);
  transition: transform 0.1s;
}

.propose-btn:active {
  transform: scale(0.95); /* クリックした時に少し凹む */
}

</style>