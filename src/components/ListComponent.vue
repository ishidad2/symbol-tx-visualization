<template>
  <div>
    <div>
      このーページでは指定したSymbolアドレス（要PublicKey）のこれまでの送信先と送信件数を調べることが出来ます。
    </div>
    <div>トータル送信件数：{{ count }} 件</div>
    <div>使用XYM：{{total_fee / 1000000}}</div>
    <ul v-for="(address, index) in addresses" :key="index">
      <li>
        送信先：<a :href="createEx(address)" target="_blank" rel="noopener noreferrer">{{ address }}</a> 件数：{{ history[address].length }} 
      </li>
    </ul>
  </div>
</template>

<script>
import { RepositoryFactoryHttp, TransactionGroup, TransactionType, Address, NetworkType } from 'symbol-sdk';
const mainnet_ex = "https://symbol.fyi/accounts/";
const testnet_ex = "https://testnet.symbol.fyi/accounts/";

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
      total_fee: 0,
      network: NetworkType.TEST_NET,
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
      this.network = await repo.getNetworkType().toPromise();
      try{
        const address =  Address.createFromPublicKey(this.publickey, this.network);
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
      const txs = await this.transactionRepo.search({
        signerPublicKey: this.publickey,
        group: TransactionGroup.Confirmed,
        pageSize: 100,
        pageNumber: this.pageNumber,
        order:"desc",
        type: [TransactionType.TRANSFER]
      }).toPromise();
      this.pageNumber++;
            
      txs.data.forEach(tx => {
        try {
          if(tx.recipientAddress.address in this.history){
            let arr = Object.assign([], this.history[tx.recipientAddress.address]);
            arr.push(tx);
            this.total_fee += tx.maxFee.compact()
            console.log(this.total_fee);
            this.history[tx.recipientAddress.address] = arr;
          }else{
            this.history[tx.recipientAddress.address] = Object.assign([], [tx]);
          }
          this.addresses.push(tx.recipientAddress.address);
          this.count++;
        } catch (error) {
          console.error(error);
        }
      });

      this.addresses = Array.from(new Set(this.addresses));
      
      if(!txs.isLastPage){
        setTimeout(() => {
          this.search();
        }, 600);
      }else{
        //重複削除
        this.$emit('setLoading', false);
        console.log('finish');
      }
      this.is_loading = false
    },
    createEx(address){
      return this.network === NetworkType.TEST_NET ? testnet_ex+address : mainnet_ex+address;
    }
  },
}
</script>