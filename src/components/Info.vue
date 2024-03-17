<template>
    <div class="info">
        <div class="detail-info">
            <div v-if="info.Name !== undefined" class="info-img">
                <img :src="info.iconHrUrl" alt="アイコン" loading="lazy">
            </div>
            <div v-if="info.Name !== undefined" class="info-name">
                {{ info.Name }}
            </div>
            <div v-if="info.Name !== undefined" class="info-clip" @click="copyToClipboard" data-bs-toggle="tooltip"
                data-bs-placement="top" data-bs-custom-class="custom-tooltip" data-offset="0, 10" title="アイテム名をコピーします。">
                <i class="far fa-copy fa-lg"></i>
            </div>
        </div>
        <div class="details">

        </div>
    </div>
</template>

<script>
import { Tooltip } from 'bootstrap'

export default {
    props: {
        info: Object,
    },
    mounted() {
        new Tooltip(document.body, {
            selector: "[data-bs-toggle='tooltip']",
        })
    },
    methods: {
        copyToClipboard() {
            const itemName = this.info.Name;
            if (navigator.clipboard) {
                navigator.clipboard.writeText(itemName)
                    .then(() => {
                        // コピー成功時の処理
                        console.log('テキストがクリップボードにコピーされました: ', itemName);
                        alert('テキストがクリップボードにコピーされました');
                    })
                    .catch(err => {
                        // クリップボードへのアクセスが拒否された場合などのエラー処理
                        console.error('クリップボードへのアクセスが拒否されました: ', err);
                        alert('クリップボードへのアクセスが拒否されました');
                    });
            } else {
                // クリップボードAPIがサポートされていない場合の処理
                console.error('クリップボードAPIがサポートされていません');
                alert('クリップボードAPIがサポートされていません');
            }
        }
    }
}
</script>

<style>
.info {
    display: flex;
    flex-direction: column;
    border-left: 2px solid #555455;
}

.detail-info {
    display: flex;
    align-items: center;
    width: 100%;
    height: 60px;
    margin: 10px;
}

.info-img {
    width: 60px;
    height: 60px;
    border-radius: 5px;
    background-color: black;
}

.info-img img {
    width: 60px;
    height: 60px;
    flex-shrink: 0;
}

.info-name {
    color: white;
    margin-left: 10px;
    font-size: 24px;
}

.info-clip {
    margin-left: 15px;
    color: white;
    cursor: pointer;
}

.details {
    width: calc(100% - 6px);
    height: calc(100vh - 195px);
    background-color: rgb(70, 95, 95, 0.3);
}
</style>
