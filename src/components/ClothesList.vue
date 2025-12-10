<script setup>
//親からデータを受け取る
const props = defineProps({
    items: Array, //itemsという名前の配列
    selectedId: Number
});

//親にイベントを伝えるための道具をつくる関数
const emit = defineEmits(['select']);

//クリックされたら親に報告する関数
const onClickItem = (id) => {
    emit('select', id);          //selectという合図と一緒にidを投げるよ
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
            <!-- item.color や item.name で中身を取り出す -->
            <div class="image-box" :style="{ backgroundColor: item.color }">
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

/* 画像代わりの箱のデザイン */
.image-box {
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: #555;
}
</style>
