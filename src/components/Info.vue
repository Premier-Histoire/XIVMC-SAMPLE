<template>
    <div v-if="info.Name !== undefined" class="detail-info">
        <div class="info-img">
            <img :src="info.iconHrUrl" alt="アイコン" loading="lazy">
        </div>
        <div class="info-name">
            {{ info.Name }}
        </div>
        <div class="info-clip" @click="copyToClipboard" data-bs-toggle="tooltip" data-bs-placement="top"
            data-bs-custom-class="custom-tooltip" data-offset="0, 10" title="アイテム名をコピーします。">
            <i class="far fa-copy fa-lg"></i>
        </div>
    </div>
    <div class="details">

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
    width: 100%;
    height: calc(100% - 111px);
    border-radius: 0 0 10px 0;
    background-color: rgb(70, 95, 95, 0.3);
}
</style>
