<template>
  <div class="full-page">
     <div class="mb-3 row">
      <div class="col-auto">
        <label for="inputPublicKey" class="col-sm-2 col-form-label">PublicKey</label>
      </div>
      <div class="col-xl-4 col-lg-6 col-md-8 col-sm-8 mb-2">
        <input type="text" class="form-control" id="inputPublicKey" v-model="publicKey">
        Address: {{ address }}
      </div>
      <div class="col-auto mb-2">
        <button class="btn btn-primary" type="button" :disabled="disabled" @click="search">
          <template v-if="disabled">
            <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
            <span class="sr-only">wait...</span>
          </template>
          <template v-else>
            Search
          </template>
        </button>
      </div>
      <div class="col-auto">
        <select class="form-select" v-model="selectNode">
          <option v-for="list in selectLists" :key="list.name" :value="list.value">{{list.name}}</option>
        </select>
      </div>
    </div>
    <div>
      <List ref="list" :publickey="publicKey" :node="selectNode" @setLoading="setLoading" @setAddress="setAddress"/>
    </div>

  </div>
</template>

<script>

import List from './components/ListComponent.vue';

export default {
  name: 'App',
  components: {
    List
  },
  data() {
    return {
      disabled: false,
      publicKey: "",
      address: null,
      selectNode: "",
      selectLists:[
        { name: "dai-testnet.sfn.tools(testnet)", value: "dai-testnet.sfn.tools" },
        { name: "dual-1.nodes-xym.work(mainnet)", value: "dual-1.nodes-xym.work" },
      ],
    }
  },
  created() {
    let urlParams = new URLSearchParams(window.location.search);
    this.$set(this, 'selectNode', (this.selectLists[Math.floor(Math.random() * this.selectLists.length)]).value);
    if(urlParams.has('node')){
      const customNode = urlParams.get('node');
      this.$set(this, "selectNode",  customNode);
      this.UnshiftArray(this.selectLists, {name:customNode, value:customNode});
    }
    if(urlParams.has('pubKey')){
      const key = urlParams.get('pubKey');
      this.$set(this, "publicKey",  key);
    }
  },
  methods: {
    UnshiftArray(array, value) {
      // 存在しない場合、配列にpushする
      if(! this.IsArrayExists(array, value)) {
        array.unshift(value);
      }
      return true;
    },
    IsArrayExists(array, value) {
      // 配列の最後までループ
      for (var i =0, len = array.length; i < len; i++) {
        if (value == array[i]) {
          // 存在したらtrueを返す
          return true;
        }
      }
      // 存在しない場合falseを返す
      return false;
    },
    search(){
      this.$refs.list.createRepo();
    },
    setLoading(bool){
      this.disabled = bool;
    },
    setAddress(address){
      this.address = address.plain();
    }
  },
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
}

.full-page {
  width: 100vw;
  height: 100vh;
  /* background: black; */
  padding: 10px;
}

.w40 {
  width: 40%;
}
</style>