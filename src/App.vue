<template>
  <div class="contents">
    <div class="search-box">
      <Search @send-data="SearchItems" />
    </div>
    <div class="result-box">
      <div class="result-header">
        <div class="result-img">
          <img v-if="searchinfo.id !== undefined" :src="`/assets/img/${this.searchinfo.id}.png`">
        </div>
        <div :class="{ 'result-text': true, 'freesearch-text': searchinfo.id === undefined }">{{ this.searchinfo.text }}
        </div>
      </div>
      <div class="result-items scroll_bar">
        <div class="result-item" v-for="(item, index) in searchResults" :key="item" @click="selectItem(item)"
          :class="{ 'last-item': index === searchResults.length - 1 }" >
          <div class="item-icon">
            <img :src="item.iconUrl" loading="lazy">
          </div>
          <div class="result-name">{{ item.Name }}</div>
          <div v-if="item.isCraftable" class="item-craftable">
            <img clss="" src="./assets/img/craft.png">
          </div>
        </div>
      </div>
    </div>
    <div class="info-box">
      <div class="info-header">
        <div class="info-text">詳細情報</div>
      </div>
      <Info :info="selectedInfo" />
    </div>
  </div>
</template>

<script>
import Search from './components/Search.vue'
import Info from './components/Info.vue'

export default {
  data() {
    return {
      isLoading: true,
      infoLoading: false,
      isSelectBoxOpen: false,
      selectedServer: '',
      selectedInfo: [],
      searchResults: [],
      searchQuery: '',
      searchinfo: [],
    }
  },
  components: {
    Search,
    Info
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
    getIconHRUrl(imageId) {
      const baseId = Math.floor(imageId / 1000) * 1000; // 1万の位を基にベースIDを算出
      const formattedImageId = imageId.toString().padStart(6, '0'); // 画像IDを6桁でフォーマット
      return `https://xivapi.com/i/0${baseId}/${formattedImageId}_hr1.png`; // 完全なURLを生成
    },
    isCraftable(itemKey) {
      return this.recipeData.some(recipe => recipe.ItemResult === itemKey);
    },
    ItemSearch(searchQuery) {
      this.selectedInfo = [],
      this.searchinfo.id = undefined;
      this.searchinfo.text = '検索';
      try {
        this.searchResults = this.itemsData.filter(item =>
          item.Name.includes(searchQuery)
        ).map(item => ({
          ...item,
          iconUrl: this.getIconUrl(item.Icon),
          iconHrUrl: this.getIconHRUrl(item.Icon),
          isCraftable: this.isCraftable(item.ItemId)
        }))
      } catch (error) {
        console.error('検索エラー:', error);
      }
    },
    FilterSearch(typeId, data, text, level, job) {
      this.selectedInfo = [],
      this.searchinfo.id = data;
      this.searchinfo.text = text
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
          iconHrUrl: this.getIconHRUrl(item.Icon),
          isCraftable: this.isCraftable(item.ItemId)
        }));
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
    selectItem(item){
      this.selectedInfo = item
      console.log(this.selectedInfo)
    }
  }
}
</script>

<style scoped>
.contents {
  width: 100%;
  height: 100%;
  background-color: #313131;
  border-radius: 10px;
  display: flex;
}

.search-box {
  width: 300px;
  padding: 0 12.5px 0 12.5px;
  border-right: 2px solid #555455;
}

.modal-body {
  height: 70vh;
}

.result-box {
  flex: 2;
  display: flex;
  align-items: center;
  flex-direction: column;
  overflow-y: auto;
  border-right: 2px solid #555455;
}

.result-header {
  width: 100%;
  height: 31px;
  background: repeat url("./assets/img/header.png");
  display: flex;
}

.result-img img {
  width: 20px;
  height: 20px;
  margin: auto 10px auto 10px;
}

.result-text {
  margin: auto 0 auto 0;
  font-size: 15px;
  color: #CFB47A;
}

.freesearch-text {
  margin-left: 40px;
}

.result-items {
  width: calc(100% - 15px);
  height: calc(100% - 31px);
  padding: 8px 10px 10px 10px;
  margin-right: 10px;
  cursor: pointer;
}

.result-item {
  display: flex;
  width: 100%;
  align-items: center;
  margin-bottom: 10px;
}

.last-item {
  margin-bottom: 0;
}

.item-icon {
  width: 40px;
  height: 40px;
  margin-right: 10px;
  background-color: black;
  border-radius: 5px;
}

.result-name {
  margin: 0;
  flex: 1;
  display: flex;
  align-items: center;
  color: white;
}

.item-craftable {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
}

.info-box {
  flex: 3;
}

.info-header {
  width: 100%;
  height: 31px;
  background: repeat url("./assets/img/header.png");
  border-radius: 0 5px 0 0;
  display: flex;
}

.info-text {
  margin: auto 0 auto 0;
  font-size: 15px;
  color: #CFB47A;
  margin-left: 30px;
}
</style>
