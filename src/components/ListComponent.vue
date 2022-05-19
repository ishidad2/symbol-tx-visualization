<template>
  <div>
    <div>
      このーページでは指定したアドレス（要PublicKey）の送信先と送信件数を調べることが出来ます。
    </div>
    <div>トータル送信件数：{{ count }} 件</div>
    <ul v-for="(address, index) in addresses" :key="index">
      <li>
        送信先：<a :href="'https://symbol.fyi/accounts/'+address" target="_blank" rel="noopener noreferrer">{{ address }}</a> 件数：{{ history[address].length }} 
      </li>
    </ul>
  </div>
</template>

<script>
import { RepositoryFactoryHttp, TransactionGroup, TransactionType, Address } from 'symbol-sdk';

const protocol = "https://";
const port = ":3001";
export default {
  props:{
    publickey: {
      type: String,
      default: "",
    },
    node: {
      type: String,
      default: "dai-testnet.sfn.tools",
    }
  },
  data() {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
      transactionRepo: null,
      pageNumber: 0,
      history: [],
      addresses: [],
      is_loading: false,
      count: 0,
    }
  },
  methods: {
    init(){
      this.pageNumber = 0;
      this.is_loading = false;
      this.count = 0;
      this.history = [];
      this.addresses = [];
      this.$emit('setLoading', true);
    },
    async createRepo(){
      if(!this.node || this.publickey === "") return;
      this.init();
      const repo = new RepositoryFactoryHttp(protocol + this.node + port);
      this.transactionRepo = repo.createTransactionRepository();
      const network = await repo.getNetworkType().toPromise();
      try{
        const address =  Address.createFromPublicKey(this.publickey, network);
        this.$emit('setAddress', address);
      }catch(e){
        this.$swal('Not the correct Symbol address.');
        this.$emit('setLoading', false);
      }
      console.log('start');
      this.search();
    },
    async search(){
      if(this.is_loading) return;
      this.is_loading = true;
      this.pageNumber++;
      const txs = await this.transactionRepo.search({
        signerPublicKey: this.publickey,
        group: TransactionGroup.Confirmed,
        pageSize: 30,
        pageNumber: this.pageNumber,
        order:"desc",
        type: [TransactionType.TRANSFER]
      }).toPromise();
      
      txs.data.forEach(tx => {
        if(tx.recipientAddress.address in this.history){
          let arr = Object.assign([], this.history[tx.recipientAddress.address]);
          arr.push(tx);
          this.history[tx.recipientAddress.address] = arr;
        }else{
          this.history[tx.recipientAddress.address] = Object.assign([], [tx]);
        }
        if(tx.recipientAddress.address === undefined){
          console.log(tx);
        }
        this.addresses.push(tx.recipientAddress.address);
        this.count++;
      });

      this.addresses = Array.from(new Set(this.addresses));
      
      if(!txs.isLastPage){
        setTimeout(() => {
          this.search();
        }, 1000);
      }else{
        //重複削除
        this.$emit('setLoading', false);
        console.log('finish');
      }
      this.is_loading = false
    },
  },
}
</script>