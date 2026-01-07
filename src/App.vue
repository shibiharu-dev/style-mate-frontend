
<script setup>
// 服を表示するコンポーネントをインポートする
// ./は同じフォルダ内という意味
import ClothesList from './components/ClothesList.vue';
//状態を記憶するためにref,onMountedという特別な関数を使う
import { ref,onMounted } from 'vue';

// -------------------トップページ部分----------------------

//選択中のIDを記憶する箱　初期値はnull
const selectedTopsId = ref(null);
const selectedBottomsId = ref(null);

// 編集モードの状態（trueなら編集ボタン表示）
const isEditMode = ref(false);

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

// -------------------服登録モーダル部分----------------------

//服登録モーダルを表示するかどうかを記憶する箱
const isModalOpen = ref(false); 
// 登録処理中かどうか
const isRegistering = ref(false);

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
  if (isRegistering.value) return; // 連打防止
  isRegistering.value = true;      // 送信開始

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

    //服を更新する
    fetchClothes();            

  } catch (error) {
    console.error("登録エラー:", error);
  } finally {
    isRegistering.value = false; // 処理終了（成功しても失敗しても実行される）
  }
};

// ----------------コーデ提案モーダル部分------------------

// ユーザーが選んだ服のデータを受け取る箱（バックエンドができたら消す）
const userSelectedCloth = ref(null);
// 提案された服のデータを受け取る箱
const suggestedCloth = ref(null); 
// 結果モーダルの開閉フラグ
const isSuggestModalOpen = ref(false); 
// 通信中フラグ（ローディング用）
const isSuggesting = ref(false); 


// バックエンドがまだないので、一時的にランダムで提案する関数
/*
const makeProposal = () => {
  // ちゃんと服が選ばれているかチェック
  if (!selectedTopsId.value && !selectedBottomsId.value) {
    alert("トップスかボトムス、どちらかを選んでください");
    return;
  }

  // ユーザーが選んだ服のデータを特定して保存
  if (selectedTopsId.value) {
    // topsDataの中から、IDが一致するものを探す
    // .find()で()内の条件にあうものを探すことができる
    userSelectedCloth.value = topsData.value.find(t => t.id === selectedTopsId.value);
  } else {
    // bottomsDataの中から探す
    userSelectedCloth.value = bottomsData.value.find(b => b.id === selectedBottomsId.value);
  }

  isSuggesting.value = true; // ローディング開始

  // サーバーとの通信のふりをして、少し待ってから結果を出す
  setTimeout(() => {
    let targetList = [];
    
    // トップスを選んでいたら、ボトムスリストから選ぶ
    if (selectedTopsId.value) {
      targetList = bottomsData.value;
    } else {
      // ボトムスを選んでいたら、トップスリストから選ぶ
      targetList = topsData.value;
    }

    // ランダムに1つ選ぶ
    // Math.floor:小数点以下切り捨て
    // Math.random():0~0.9999まででランダムに小数を選ぶ
    const randomIndex = Math.floor(Math.random() * targetList.length);
    const randomCloth = targetList[randomIndex];

    suggestedCloth.value = randomCloth; // 結果を箱に入れる
    isSuggestModalOpen.value = true;    // モーダルを開く
    isSuggesting.value = false;         // 通信終了
  }, 1000); // 1000ミリ秒 = 1秒待つ
};
*/

// 本来のAPIを使う関数（バックエンドができたらこっちに戻す）

const makeProposal = async () => {
  // 1. ちゃんと服が選ばれているかチェック
  if (!selectedTopsId.value && !selectedBottomsId.value) {
    alert("トップスかボトムス、どちらかを選んでください");
    return;
  }

  // 選択された服のデータを取得
  let selectedCloth = null;
  if (selectedTopsId.value) {
    selectedCloth = topsData.value.find(t => t.id === selectedTopsId.value);
  } else {
    selectedCloth = bottomsData.value.find(b => b.id === selectedBottomsId.value);
  }

  // 画面表示用にセット
  userSelectedCloth.value = selectedCloth;
  isSuggesting.value = true; // ローディング開始

  try {
    // 3. サーバーに送信 (POST)
    // URLの末尾に選んだ服のIDをつける
    
    //    const response = await fetch(`http://localhost:5000/api/suggest/${selectedCloth.id}`, {
    //  method: 'POST'

    const response = await fetch(`http://localhost:5000/api/suggest/${selectedCloth.id}`, {
      method: 'POST'
    });

    // 4. 結果を受け取る
    // 処理に時間がかかるのでawaitをつける
    // .json() を使ってJavaScriptで扱えるデータに戻す
    const data = await response.json();
    
    // 5. 結果を表示
    // レスポンスには提案された服のデータが入っている
    // 手持ちのリストから探して、名前などの情報を補完する
    const allClothes = [...topsData.value, ...bottomsData.value];
    const bestMatch = allClothes.find(item => item.id === data.id);

    if (bestMatch) {
      suggestedCloth.value = bestMatch;
    } else {
      // リストに見つからない場合は、レスポンスのデータをそのまま使う
      suggestedCloth.value = {
        ...data,
        name: 'AIのおすすめ'
      };
    }

    isSuggestModalOpen.value = true; // モーダルを開く

  } catch (error) {
    console.error("提案エラー:", error);
    alert("AIの提案に失敗しました...");
  } finally {
    isSuggesting.value = false; // 通信終了（成功しても失敗しても実行される）
  }
};



//-----------------データ受け取り-----------------

// 非同期関数 (async) として定義
const fetchClothes = async () => {
  try {
    // 1. サーバーに「データちょうだい」と言う
    const response = await fetch('http://localhost:5000/api/clothes');
    
    // 2. 返ってきたデータをJSONとして読み込む
    const data = await response.json();
    
    // 3. データをトップスとボトムスに振り分ける
    // (サーバーからは全データのリストが返ってくる想定)
    topsData.value = data.filter(item => item.genre === 'Top');
    bottomsData.value = data.filter(item => item.genre === 'Bottom');
  } catch (error) {
    console.error("データの取得に失敗しました:", error);
  }
};

// ----------------データ部分------------------

const topsData = ref([]);
const bottomsData = ref([]);


onMounted(() => {
  fetchClothes();
});

// 画像のURLを取得する関数
const getImageUrl = (path) => {
  if (!path) return '';
  // Windowsパスの修正
  let cleanPath = path.replace(/\\/g, '/');
  // すでにhttpならそのまま
  if (cleanPath.startsWith('http')) return cleanPath;

  const baseUrl = 'http://localhost:5000';
  // パスの先頭に / がなければつける
  return `${baseUrl}${cleanPath.startsWith('/') ? '' : '/'}${cleanPath}`;
};

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
        <ClothesList :items="topsData" :selected-id="selectedTopsId" :is-edit-mode="isEditMode" @select="handleTopsSelect"/>
      </div>

      <!--ボトムスエリア-->
      <div class="clothing-section">
        <h2>ボトムス</h2>
        <!-- :items="bottomsData" でデータを渡す！ -->
        <ClothesList :items="bottomsData" :selected-id="selectedBottomsId" :is-edit-mode="isEditMode" @select="handleBottomsSelect"/>
      </div>
      
      <!-- 提案ボタン-->
      <div class="action-area" v-if="selectedTopsId || selectedBottomsId">
        <button class="propose-btn" @click="makeProposal">
          コーディネートを提案する！
        </button>
      </div>
      
      <!-- 編集モード切り替えボタン（左下） -->
      <button class="fab-btn edit-mode-toggle" @click="isEditMode = !isEditMode" :class="{ active: isEditMode }">
        <!-- 鉛筆アイコン -->
        <svg xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 -960 960 960" width="24" fill="white"><path d="M200-200h57l391-391-57-57-391 391v57Zm-80 80v-170l528-527q12-11 26.5-17t30.5-6q16 0 31 6t26 17l55 56q12 11 17.5 26t5.5 30q0 16-5.5 30.5T817-647L290-120H120Zm635-577-57-57 57 57Zm-58 59L258-120H120v-138l427-427 138 138Z"/></svg>
      </button>

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
            <button @click="registerCloth" class="register-btn" :disabled="isRegistering">
              {{ isRegistering ? '送信中...' : '登録' }}
            </button>
            <button @click="isModalOpen = false" class="cancel-btn" :disabled="isRegistering">キャンセル</button>
          </div>
        </div> 
      </div>

      <!-- AIが提案した服を表示するモーダル -->
      <div v-if="isSuggestModalOpen" class="modal-overlay">
        <div class="modal-content suggest-modal">
          <h3>AIのおすすめコーデ</h3>
          
          <!-- 結果表示部分 -->
          <div class="result-area">
            <!-- 自分が選んだ服 -->
            <div class="cloth-card">
              <p>あなたの選択</p>
              <!-- 画像の代わりに色の箱を表示 -->
              <!-- :styleでcolor-boxのcssの内容を変えられる-->
              <!-- userSelectedCloth?.colorのように?.にすると、データがない時は何もしない（undefined）を渡してくれる-->
              <!-- ngrokを使わない時ははgetImageUrlはいらない -->
              <img v-if="userSelectedCloth?.image_path" :src="getImageUrl(userSelectedCloth.image_path)" class="result-img" alt="選択した服">
              <div v-else class="color-box" :style="{ backgroundColor: userSelectedCloth?.color }"></div>
              <p>{{ userSelectedCloth?.name }}</p>
            </div>
            
            <div class="plus-icon">＋</div>

            <!-- AIが選んだ服 -->
            <div class="cloth-card">
              <p>AIの提案</p>
              <!-- 画像の代わりに色の箱を表示 -->
              <!-- ngrokを使わない時ははgetImageUrlはいらない -->
              <img v-if="suggestedCloth?.image_path" :src="getImageUrl(suggestedCloth.image_path)" class="result-img" alt="提案された服">
              <div v-else class="color-box" :style="{ backgroundColor: suggestedCloth?.color }"></div>
              <p>{{ suggestedCloth?.name }}</p>
            </div>
          </div>

          <button class="close-btn" @click="isSuggestModalOpen = false">閉じる</button>
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

/* 編集モードボタン（左下） */
.edit-mode-toggle {
  right: auto; /* 右寄せを解除 */
  left: 20px;  /* 左に配置 */
  background-color: #888; /* 通常時はグレー */
}

/* 編集モードONのとき */
.edit-mode-toggle.active {
  background-color: #f0ad4e; /* ONのときはオレンジ */
  transform: scale(1.1);     /* 少し大きくする */
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

/* 服登録モーダル*/
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
.register-btn:disabled {
  background-color: #ccc; /* 無効時はグレーにする */
  cursor: not-allowed;    /* マウスカーソルを禁止マークに */
}
.cancel-btn {
  background-color: #888585; 
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
}

/* コーデ提案モーダル */
.result-area {
  display: flex;           /* 中身を横並びにする魔法 */
  align-items: center;     /* 上下の真ん中に揃える */
  justify-content: center; /* 左右の真ん中に揃える */
  gap: 15px;               /* アイテム同士の隙間を15px空ける */
  margin: 20px 0;          /* 外側の余白（上下に20px、左右は0） */
}
.cloth-card {
  text-align: center;      /* 文字や中身を中央揃えにする */
  width: 100px;            /* カードの幅を100pxに固定 */
}
.color-box {
  width: 80px;             /* 箱の幅 */
  height: 80px;            /* 箱の高さ */
  border-radius: 8px;      /* 角を少し丸くする */
  margin: 5px auto;        /* 上下に5pxの隙間、左右は自動で中央寄せ */
  border: 1px solid #ddd;  /* 薄いグレーの枠線をつける */
}
.result-img {
  width: 80px;
  height: 110px;
  border-radius: 8px;
  object-fit: cover;
  margin: 5px auto;
}
.plus-icon {
  font-size: 24px;         /* ＋マークの文字サイズ */
  color: #888;             /* グレー色にする */
}
.close-btn{
  background-color: #42b883; /* ボタンの背景色（緑） */
  color: white;              /* 文字色を白に */
  border: none;              /* 枠線を消す */
  padding: 10px 20px;        /* 内側の余白（上下10px、左右20px） */
  border-radius: 5px;        /* 角を丸くする */
  cursor: pointer;           /* マウスを乗せたときに指マークにする */
  display: block;   /* ブロック要素にして幅を確保 */
  margin: 0 auto;   /* 左右の余白を自動にして中央寄せ */
}
</style>