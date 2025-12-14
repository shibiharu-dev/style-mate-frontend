
<script setup>
// 服を表示するコンポーネントをインポートする
// ./は同じフォルダ内という意味
import ClothesList from './components/ClothesList.vue';
//状態を記憶するためにrefという特別な関数を使う
import { ref } from 'vue';

//選択中のIDを記憶する箱　初期値はnull
const selectedTopsId = ref(null);
const selectedBottomsId = ref(null);

//服登録モーダルを表示するかどうかを記憶する箱
const isModalOpen = ref(false); 

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
  console.warn("選ばれたボトムスのID:", id);
};

//提案するボタンがクリックされたときに動く関数
const makeProposal = () => {
  alert(`ID:${selectedTopsId.value} と ID:${selectedBottomsId.value} でコーディネートしました！`);
};

//ユーザーが入力したデータを一時的に保存しておく箱
const newCloth = ref({
  genre: 'Top', // 初期値
  color: '#000000',
  image: null   // ファイルは最初は空
});

//画像ファイルを受け取る関数
const handleFileChange = (event) => {
  // event.target.files は選ばれたファイルのリスト
  // [0] でその中の1つ目（ユーザーが選んだ画像）を取り出す
  const file = event.target.files[0];
  if (file) {
    //newClothのimageに画像ファイルを入れる
    newCloth.value.image = file;
  }
};

const registerCloth = async () => {
  // 1. 送るデータを作る
  // 画像データを含むデータを送るときはFormData形式を使う
  const formData = new FormData();
  formData.append('image', newCloth.value.image);
  formData.append('genre', newCloth.value.genre);
  formData.append('color', newCloth.value.color);

  try {
    // 2. サーバーに送信 (POSTメソッド)
    // await は「送信が終わるまでここで待つ」という意味
    await fetch('http://localhost:5000/api/clothes', {
      method: 'POST',
      body: formData,
    });
    
    // 3. 成功したら...
    //モーダルを閉じて、服リストを更新する
    isModalOpen.value = false; 
    // fetchClothes();            
    alert("登録しました！");

  } catch (error) {
    console.error("登録エラー:", error);
  }
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
  { id: 5, name: "スカート", color: "ffaaaa"},
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
      
      <!-- 登録ボタン -->
      <!-- クリックするとisModalOpenがtrueになる -->
      <button class="fab-btn" @click="isModalOpen = true">＋</button> 
      
      <!-- モーダル　-->
      <!-- v-if="isModalOpen"：もし isModalOpen が true なら、この要素を表示する-->
      <div v-if="isModalOpen" class="modal-overlay"> 
        <div class="modal-content">
          <h3>新しい服を登録</h3>
          
          <!-- 入力フォーム部分-->
          <!--  v-modelを使って、画面の入力と変数（newCloth）をリアルタイムで同期させる-->
          
          <!-- 画像アップロード -->
          <!-- 画像ファイルはv-modelを使えないので、@changeを使って、変更があったときにhandleFileChangeを呼び出して保存する-->
          <!-- accept="image/*" は画像だけを選べるようにする　image(画僧ファイル)/*()-->
          <div class="form-group">
            <label>画像</label>
            <input type="file" @change="handleFileChange" accept="image/*">
          </div>

          <!-- ジャンル選択 -->
          <!-- トップスを選ぶとnewCloth.genreにTopが入力される -->
          <div class="form-group">
            <label>ジャンル</label>
            <select v-model="newCloth.genre">
              <option value="Top">トップス</option>
              <option value="Bottom">ボトムス</option>
            </select>
          </div>
          
          <!-- 色選択 -->
          <!-- HTML5（Webの標準ルール）の機能をつかって色を選択する-->
          <div class="form-group">
            <label>色</label>
            <input type="color" v-model="newCloth.color" class="color-picker">
          </div>
          
          <!-- 登録ボタンとキャンセルボタン -->
          <!-- 登録をおすと、registerCloth 関数が動き、サーバーへの送信処理などが始まる-->
          <!-- キャンセルを押すと、isModalOpen を false に戻す。v-if の条件が外れるため、モーダルが消えます。-->
          <div class="modal-actions">
            <button @click="registerCloth" class="register-btn">登録</button>
            <button @click="isModalOpen = false" class="cancel-btn">キャンセル</button>
          </div>
        </div> 
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
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

/*h2*/
h2 {/*注意ここには.をつけない*/
  margin-bottom: 10px;
  font-size: 1.2rem;
  border-bottom: 2px solid #ddd; /* 下線 */
}

/* モーダルのタイトル */
h3 {
  text-align: center;  /* 文字を真ん中に */
  margin-bottom: 15px; /* 下との間隔を広げる */
}


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

/* 右下の＋ボタン */
.fab-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 60px;
  height: 60px;
  border-radius: 50%; /* 角を丸くする */
  background-color: #42b883;
  color: white;
  font-size: 30px;
  border: none; /* 枠線を消す */
  box-shadow: 0 4px 10px rgba(0,0,0,0.3);
  cursor: pointer;
  z-index: 100; /*他のものと重なっても隠れないように*/
}

/* モーダルの背景 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 200; /*ボタンよりも値が大きいので、ボタンよりもさらに手前に表示される*/
}

/* モーダルの中身 */
.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  width: 90%; /*画面の幅の90%まで広げる*/
  max-width: 400px; /*ただし、400pxまで*/
}

/* フォームのスタイル */
.form-group {
  margin-bottom: 15px; /* 下の項目との間隔 */
  text-align: center;  /* 中身を中央揃え */
}
.form-group label {
  display: block;      /* ブロック要素にして改行させる（入力欄の上に表示） */
  margin-bottom: 5px;  /* 入力欄との隙間 */
  font-weight: bold;   /* 太字 */
  color: #333;
}
.form-group select, .form-group input[type="file"] {
  width: 100%;         /* 横幅いっぱいに広げる */
  padding: 8px;        /* 内側の余白 */
  border: 1px solid #ddd; /* 薄いグレーの枠線 */
  border-radius: 4px;  /* 角を少し丸く */
}
.color-picker {
  width: 100%;
  height: 40px;        /* 押しやすい高さに */
  cursor: pointer;     /* マウスを乗せたら指マークに */
  border: none;
}
.modal-actions {
  display: flex;       /* ボタンを横並びにする */
  justify-content: space-between; /* ボタンを左右の端に離して配置 */
  margin-top: 20px;
}
.register-btn {
  background-color: #42b883; /* 緑色 */
  color: white;
  border: none;
  padding: 10px 20px;  /* ボタンの大きさ */
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
}
.cancel-btn {
  background-color: #888585; 
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
}
</style>