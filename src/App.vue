<template>
  <div class="contents">
    <div class="search-box">
      <Search @send-data="SearchItems" />
    </div>
    <div class="result-box">
      <div v-for="item in searchResults" :key="item" @click="selectItem(item)">
        <img :src="item.iconUrl" alt="アイコン" class="item-icon" loading="lazy">
        <div class="item-info">
          <div class="item-name">{{ item.Name }}</div>
          <div v-if="item.isCraftable" class="item-craftable"><img class="craft"
              src="https://xivapi.com/cj/1/blacksmith.png"></div>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered modal-xl">
      <div class="modal-content">
        <div class="modal-header">
          <h1 class="modal-title fs-5" id="exampleModalLabel">詳細</h1>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          ...
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Search from './components/Search.vue'

export default {
  data() {
    return {
      isLoading: true,
      infoLoading: false,
      isSelectBoxOpen: false,
      selectedServer: '',
      selectedInfo: null,
      searchResults: [],
      searchQuery: '',
    }
  },
  components: {
    Search
  },
  created() {
    this.loadJsonData();
  },
  methods: {
    async loadJsonData() {
      try {
        this.isLoading = true;
        const itemsResponse = await fetch('/json/Item.json');
        const recipeResponse = await fetch('/json/Recipe.json');
        const classJobCategoriesResponse = await fetch('/json/ClassJobCategory.json');
        if (!itemsResponse.ok || !recipeResponse.ok || !classJobCategoriesResponse.ok) {
          throw new Error('JSONファイルの読み込みに失敗しました。');
        }
        this.itemsData = await itemsResponse.json();
        this.recipeData = await recipeResponse.json();
        this.classJobCategories = await classJobCategoriesResponse.json();
      } catch (error) {
        console.error('JSON読み込みエラー:', error);
      } finally {
        this.isLoading = false;
      }
    },
    SearchItems(id, typeId, data, level, job) {
      if (id === 'Category') {
        this.FilterSearch(typeId, data, level, job)
      } else {
        this.ItemSearch(data)
      }
    },
    getIconUrl(imageId) {
      const baseId = Math.floor(imageId / 1000) * 1000; // 1万の位を基にベースIDを算出
      const formattedImageId = imageId.toString().padStart(6, '0'); // 画像IDを6桁でフォーマット
      return `https://xivapi.com/i/0${baseId}/${formattedImageId}.png`; // 完全なURLを生成
    },
    isCraftable(itemKey) {
      return this.recipeData.some(recipe => recipe.ItemResult === itemKey);
    },
    ItemSearch(searchQuery) {
      try {
        this.searchResults = this.itemsData.filter(item =>
          item.Name.includes(searchQuery)
        ).map(item => ({
          ...item,
          iconUrl: this.getIconUrl(item.Icon),
          isCraftable: this.isCraftable(item.ItemId)
        }));
        console.log(this.searchResults);
      } catch (error) {
        console.error('検索エラー:', error);
      }
    },
    FilterSearch(typeId, data, level, job) {
      console.log(this.itemsData);
      let selectedJobId; // selectedJobId を定義する

      try {
        if (job !== undefined) {
          selectedJobId = this.findClassJobId(job); // selectedJobId を設定する
        }
        this.searchResults = this.itemsData.filter(item => {
          if (typeId === 1 || typeId === 2) {
            return parseInt(item.ItemSearchCategory) === data &&
              ((parseInt(item.LevelEquip) <= level) ||
                (parseInt(item.LevelEquip) <= level && selectedJobId.includes(String(item.ClassJobCategory)))
              );
          } else {
            return parseInt(item.ItemSearchCategory) === data;
          }
        }).map(item => ({
          ...item,
          iconUrl: this.getIconUrl(item.Icon),
          isCraftable: this.isCraftable(item.ItemId)
        }));
        console.log(this.searchResults);
      } catch (error) {
        console.error('検索エラー:', error);
      }
    },

    findClassJobId(selectedJob) {
      const matchingIds = [];
      for (const category of this.classJobCategories) {
        if (category[selectedJob] === "TRUE") {
          matchingIds.push(category.ID);
        }
      }
      return matchingIds;
    },
  }
}
</script>

<style scoped>
.contents {
  width: 100%;
  height: 100%;
  background-color: #313031;
  border: 1px solid black;
  border-radius: 10px;
  display: flex;
}

.search-box {
  width: 300px;
  margin: 2.5px;
  border-radius: 5px 0 0 5px;
  padding: 0 10px 0 10px;
  border-right: 1px solid white;
}

.result-box {
  flex: 1;
  margin: 2.5px;
  border-radius: 0 5px 5px 0;
}

.modal-body {
  height: 70vh;
}

/* 検索結果ボックス */
.result-box {
  display: flex;
  flex-direction: column; /* 縦の行並びにする */
  width: 700px;
  max-height: 400px; /* 最大の高さを設定し、高さが超えた場合にスクロールする */
  overflow-y: auto; /* 縦方向のスクロールバーを表示する */
}

/* アイテム */
.item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  cursor: pointer;
}

/* アイコン */
.item-icon {
  width: 50px;
  height: 50px;
  margin-right: 10px;
}

/* アイテム情報 */
.item-info {
  display: flex;
  flex-direction: row; /* アイコンとテキストを横に並べる */
  align-items: center; /* 縦方向の中央揃え */
}

/* クラフトアイコン */
.craft {
  width: 20px;
  height: 20px;
  margin-left: 5px; /* アイコンとテキストの間に適切な間隔を設定 */
}

/* アイテム名 */
.item-name {
  font-weight: bold;
}

/* クラフト可能アイコン */
.item-craftable {
  margin-top: 5px;
}


</style>
