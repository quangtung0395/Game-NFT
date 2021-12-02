<template>
  <div class="hello">
    <div class="container-fluid">
      <div class="tt-noidung">
        <img src="/images/loading.gif" alt="" v-if="loading">
        <div v-if="showHide">
          <h4>NHÂN VẬT HIỆN CÓ: <span style="color: red">{{dataArr.length}}</span> </h4>
          <h4 v-if="showNoNhanVat">Không tìm thấy kết quả nào phù hợp!</h4>
          <button type="button" class="tt-checkchiso" style="margin-bottom: 30px" v-on:click="checkAllMetamon()">Check chỉ số tất cả metamon</button>
          <button type="button" value="number" class="tt-checkchiso" style="margin-bottom: 30px" v-on:click="filterTotalPower()">Filter By Total Power</button>
          <button type="button" value="number" class="tt-checkchiso" style="margin-bottom: 30px" v-on:click="filterTotalLevel()">Filter By Level</button>
          
          <h4>Check chỉ số trước khi FILTER</h4>
          <!-- <button type="button" class="tt-checkchiso" style="margin-bottom: 30px" v-on:click="filterByLevel()">Filter By Level</button> -->
          <div class="row" id="id01">
            <div class="tt-item111 col-md-6"  v-for="item in dataArr" :key="item.id" :id="'tt-item111-'+item.token_id">
              <div class="tt-item">
                <div class="row">
                  <div class="col-md-6">
                    <div class="row">
                      <div class="col-md-4">
                        <div class="tt-item-img">
                          <img :src="item.image_url" alt="">
                        </div>
                      </div>
                      <div class="col-md-8">
                        <div class="tt-item-text">
                          <ul>
                            <li><span>Tên Nhân Vật:</span> {{item.name}}#{{item.token_id}}</li>
                            <li><span>Đơn giá: </span> <strong v-html="formatNumber(item.fixed_price * item.count)"></strong> Raca</li>
                            <li><span>Số Lượng: </span>{{item.count}}</li>
                            <li><span>Tổng Tiền: </span> <strong v-html="formatNumber(item.fixed_price)"></strong> Raca</li>
                            <li><a :href="'https://market.radiocaca.com/#/market-place/'+item.id" target="_blank">Đến trang mua</a></li>
                          </ul>
                        </div>
                      </div>
                    </div>
                  </div>
                  <div class="col-md-6">
                    <button type="button" class="has_mt check-meta tt-checkchiso" :id="item.token_id" v-on:click="checkChiSo(item.token_id,item.id)">Check chỉ số</button>
                    <img src="/images/loading.gif" alt="" v-if="loadingChiSo">
                    <div :class="'flex tt-checkchiso-content metadata-' + item.token_id"></div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import $ from 'jquery'
export default {
  data() {
    return {
      dataArr: null,
      dataArr1: null,
      tokenId: [],
      nameSearch: '',
      input2: '',
      loading: false,
      showHide: false,
      showNoNhanVat: false,
      showCanQuetMoi: false,
      attributesObject: {},
      numberTotalPower: 0,
      loadsort: false,
      loadingChiSo: false
    }
  },
  methods: {
    onChangeTextSearch(event) {
      // this.nhanvat(event)
      this.nameSearch = event
    },
    async nhanvat() {
      // GET DATA nhân lực
      this.loading = true
      var dataNhanvat
      await axios.get('https://market-api.radiocaca.com/nft-sales?pageNo=1&pageSize=100&sortBy=created_at&order=desc&name=&saleType&category=13&tokenType')
        .then(response => {
          dataNhanvat = response.data.list
        })
      this.dataArr = dataNhanvat
      // console.log('dataNhanvat', dataNhanvat)
      this.showCanQuetMoi = true
      this.loading = false
      this.showHide = true
    },
    async checkChiSo(tokenId,iditem) {
        // console.log('index', iditem)
        this.loadingChiSo = true
        document.getElementById(tokenId).style.display = "none";
        //   // GET DATA CÔNG THỦ
        var owner
        await axios.get('https://market-api.radiocaca.com/nft-sales/'+ iditem)
        .then(response => {
          // console.log('nft-sales', response)
            if (response.data) {
              if(response.data.code === 200){
                // console.log('response.data', response.data)
                owner = response.data.data.owner
                // console.log('owner', owner)
              }
            }
        })


        var idArtwork
        await axios.get('https://market-api.radiocaca.com/artworks?pageNo=1&pageSize=100&address=' + owner + '&status=on_sale')
        .then(response => {
          // console.log('response ownw', response)
          var dataResOwnerArr = response.data.list
          for(var i = 0; i < dataResOwnerArr.length; i++){
            // console.log( 'Token'+ [i],dataResOwnerArr[i].token_id, tokenId)
            if( parseInt(dataResOwnerArr[i].token_id) ===  parseInt(tokenId) ){
              // console.log( 'iditem',dataResOwnerArr[i].token_id, tokenId)
              idArtwork = dataResOwnerArr[i].id
            }
          }
        })

        await axios.get('https://market-api.radiocaca.com/artworks/' + idArtwork)
        .then(response => {
          // console.log('response artworks', response)
            var metadataText = '';
            var totalPower = 0;
            var totalLevel = 0;
            var artworksData = response.data.data.properties
            metadataText += "<div class='row'>"
            for (var i =0; i<artworksData.length; i++){
              if (artworksData[i].key === 'Score'){
                totalPower = artworksData[i].value
              }
              if (artworksData[i].key === 'Level'){
                totalLevel = artworksData[i].value
              }
              metadataText += "<div class='col-6'> <strong>"+ artworksData[i].key + "</strong>: " + artworksData[i].value + "</div>";
            }
            metadataText = "<div class='col-6'>Total power: <strong class='sortTotalPower' token-id = '"+ tokenId +"' data-numbers ='" + totalPower +"'>" + totalPower + "</strong></div>" + metadataText;
             metadataText = "<div class='col-6'>Level: <strong class='sortTotalLevel' token-id = '"+ tokenId +"' data-numbers-level ='" + totalLevel +"'>" + totalLevel + "</strong></div>" + metadataText;
            metadataText += "</div>"
            document.querySelector('.metadata-' + tokenId).innerHTML = metadataText;
        })
        this.loadingChiSo = false

    },
    checkAllMetamon() {
        var buttons = document.querySelectorAll('.check-meta');
        buttons.forEach((button) => {
            button.click();
        });
    },
    async filterTotalPower() {
      var divList = $(".tt-item111");
      var numberTotalPower = $(".sortTotalPower")
      for (var i = 0; i< numberTotalPower.length; i++){
        if(numberTotalPower[i].getAttribute('data-numbers') != 0){
          $("#tt-item111-"+numberTotalPower[i].getAttribute('token-id')).attr('data-number', numberTotalPower[i].getAttribute('data-numbers'));
        } else {
          $("#tt-item111-"+numberTotalPower[i].getAttribute('token-id')).attr('data-number', '0');
        }
        console.log('numberTotalPower'+ [i], numberTotalPower[i].getAttribute('data-numbers'))
      }
      divList.sort(function(a, b) {
          return parseInt($(b).data('number')) - parseInt($(a).data('number'));
      });
      $("#id01").html(divList);
    },
    async filterTotalLevel() {
      var divList = $(".tt-item111");
      var numberTotalPower = $(".sortTotalLevel")
      for (var i = 0; i< numberTotalPower.length; i++){
        if(numberTotalPower[i].getAttribute('data-numbers-level') != 0){
          $("#tt-item111-"+numberTotalPower[i].getAttribute('token-id')).attr('data-numberlevel', numberTotalPower[i].getAttribute('data-numbers-level'));
        } else {
          $("#tt-item111-"+numberTotalPower[i].getAttribute('token-id')).attr('data-numberlevel', '0');
        }
        console.log('numberTotalLevel'+ [i], numberTotalPower[i].getAttribute('data-numbers'))
      }
      divList.sort(function(a, b) {
          return parseInt($(b).data('numberlevel')) - parseInt($(a).data('numberlevel'));
      });
      $("#id01").html(divList);
    },
    formatNumber(num) {
        return num.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,')
    },
  },
  mounted () {
    this.nhanvat()
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.tt-item-text ul li{
  list-style-type: none;
  margin: 0;
  text-align: left;
}
.tt-item-text ul li span{
  font-weight: 600;
}
.tt-item-img img{
  width: auto;
  height: 150px;
}
.tt-search{
  margin-bottom: 30PX;
}
h4{
  font-size: 20px;
  margin-bottom: 30px;
  font-weight: 600;
  text-align: center;
}
.tt-item{
      margin-bottom: 10px;
    border-radius: 5px;
    padding: 5px 3px;
    box-shadow: 0 1px 4px 1px rgb(0 0 0 / 25%);
}
.tt-search a{
  text-align: left;
  display: block;
  margin-top: 10px;
}
.tt-checkchiso{
  background: #ffcc00;
    border: none;
    font-size: 14px;
    padding: 5px 10px;
    font-weight: 600;
    margin-right: 30px;
}
.tt-checkchiso-content{
    text-align: left;
}
</style>
