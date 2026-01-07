<script setup>
//親からデータを受け取る
const props = defineProps({
    items: Array, //itemsという名前の配列
    selectedId: Number,
    isEditMode: Boolean // 編集モードかどうかを受け取る
});

//親にイベントを伝えるための道具をつくる関数
const emit = defineEmits(['select', 'edit', 'delete']);

//クリックされたら親に報告する関数
const onClickItem = (id) => {
    emit('select', id);          //selectという合図と一緒にidを投げるよ
};

//編集ボタンが押されたら親に報告する関数
const onEditClick = (item) => {
    emit('edit', item);
};

//削除ボタンが押されたら親に報告する関数
const onDeleteClick = (item) => {
    emit('delete', item);
};

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
    <!-- 服リスト全体を囲む枠 -->
    <div class="clothes-list">

        <!-- 服アイテム  -->
         <!-- v-forはデータの中身を一つ一つ取り出す -->
         <!-- 自分のidと選ばれているidが同じならselectedクラスがつく-->
         <!-- @clickでこの要素がクリックされたときに関数を動かすようにしている-->
        <div v-for="item in items" :key="item.id" class="card" :class="{ 'selected': item.id === selectedId}" @click="onClickItem(item.id)">
            <!-- 編集ボタン -->
            <!-- @click.stop は「親のクリックイベント(onClickItem)を邪魔しない」という意味 -->
            <button v-if="isEditMode" class="edit-btn" @click.stop="onEditClick(item)">
                <!-- 鉛筆のSVGアイコン -->
                <svg xmlns="http://www.w3.org/2000/svg" height="18" viewBox="0 -960 960 960" width="18" fill="#555"><path d="M200-200h57l391-391-57-57-391 391v57Zm-80 80v-170l528-527q12-11 26.5-17t30.5-6q16 0 31 6t26 17l55 56q12 11 17.5 26t5.5 30q0 16-5.5 30.5T817-647L290-120H120Zm635-577-57-57 57 57Zm-58 59L258-120H120v-138l427-427 138 138Z"/></svg>
            </button>

            <!-- 削除ボタン（左上） -->
            <button v-if="isEditMode" class="delete-btn" @click.stop="onDeleteClick(item)">
                <!-- ゴミ箱のSVGアイコン -->
                <svg xmlns="http://www.w3.org/2000/svg" height="18" viewBox="0 -960 960 960" width="18" fill="#ff4444"><path d="M280-120q-33 0-56.5-23.5T200-200v-520h-40v-80h200v-40h240v40h200v80h-40v520q0 33-23.5 56.5T680-120H280Zm400-600H280v520h400v-520ZM360-280h80v-360h-80v360Zm160 0h80v-360h-80v360ZM280-720v520-520Z"/></svg>
            </button>

            <!-- item.color や item.name で中身を取り出す -->
            <!-- <img v-if="item.image_path" :src="item.image_path" class="cloth-image" alt="服の画像"></img> -->
            <img v-if="item.image_path" :src="getImageUrl(item.image_path)" class="cloth-image" alt="服の画像">
            <div v-else class="image-box" :style="{ backgroundColor: item.color }">
                <span>{{ item.name }}</span>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* リスト全体のスタイル */
.clothes-list {
    display: flex;        /* 子要素を横並びにする */
    overflow-x: auto;     /* 横にはみ出したらスクロールさせる */
    gap: 15px;            /* アイテム同士の隙間 */
    padding: 12px 16px;   /* 上下、左右に余白　選択時用の余白 */
    padding-bottom: 20px; /* スクロールバーのための余白 */
}

/* 各カードのスタイル */
.card {
    position: relative;   /* 子要素(編集ボタン)を絶対配置するために必要 */
    flex: 0 0 auto;       /* 勝手に縮まないようにする */
    width: 120px;         /* カードの幅 */
    border: 1px solid #ddd;
    border-radius:8px;
    overflow: hidden;     /* 角丸からはみ出た部分をカット */   
    cursor: pointer;        /* ★追加: カーソルを指マークにする */
    transition: all 0.2s;   /* ★追加: 変化をアニメーションさせる */
}

/* 選択中のスタイル */
.card.selected {
    border: 4px solid #42b883;
    transform: scale(1.03); /* カードの大きさを変形　*/
    box-shadow: 0 4px 10px rgba(66, 184, 131, 0.5); /* 影　横方向　縦方向　ぼかしの強さ　影の色 */
}

/* 画像のスタイル */
.cloth-image {
    width: 100%;
    height: 150px;
    object-fit: cover; /* 枠に合わせてトリミング */
}

/* 画像代わりの箱のデザイン */
.image-box {
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: #555;
}

/* 編集ボタンのスタイル */
.edit-btn {
    position: absolute; /* カードの右上に固定 */
    top: 5px;
    right: 5px;
    background: rgba(255, 255, 255, 0.9); /* 半透明の白背景 */
    border: 1px solid #ccc;
    border-radius: 50%; /* 丸くする */
    width: 28px;
    height: 28px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0;
    z-index: 10; /* 画像より手前に表示 */
}

/* 削除ボタンのスタイル */
.delete-btn {
    position: absolute; /* カードの左上に固定 */
    top: 5px;
    left: 5px;
    background: rgba(255, 255, 255, 0.9);
    border: 1px solid #ccc;
    border-radius: 50%;
    width: 28px;
    height: 28px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0;
    z-index: 10;
}
</style>
