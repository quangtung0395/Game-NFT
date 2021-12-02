<template>
  <div class="hello">
    <div class="container-fluid">
      <!-- <div class="row">
        <div class="col-md-3"></div>
        <div class="col-md-6">
          <div class="tt-search">
            <h4>NHẬP TÊN NHÂN VẬT</h4>
            <el-input
                placeholder="Search Tên Nhân Vật"
                suffix-icon="el-icon-search"
                v-model="input2"
                value="input2"
                v-on:change="onChangeTextSearch">
              </el-input>
            <a href="/" v-if="showCanQuetMoi">Càn quét mới</a>
          </div>
        </div>
      </div> -->
      <div class="tt-noidung">
        <img src="/images/loading.gif" alt="" v-if="loading">
        <div v-if="showHide">
          <h4>NHÂN VẬT HIỆN CÓ: <span style="color: red">{{dataArr.length}}</span> </h4>
          <h4 v-if="showNoNhanVat">Không tìm thấy kết quả nào phù hợp!</h4>
          <!-- <div class="container">
            <button class="tt-checkchiso" @click="handleSortLevel()">Sort By Level</button>
            <button class="tt-checkchiso" @click="handleSortPower()">Sort By Total Power</button>
          </div> -->
          <div class="row">
            <div class="col-md-6"  v-for="item in dataArr" :key="item.id">
              <div class="tt-item">
                <div class="row">
                  <div class="col-md-2">
                    <div class="tt-item-img">
                      <img :src="item.image" alt="">
                    </div>
                  </div>
                  <div class="col-md-4">
                    <div class="tt-item-text">
                      <ul>
                        <li><span>Tên Nhân Vật:</span> {{item.name}}#{{item.token_id}}</li>
                        <li><span>Đơn giá: </span>{{item.fixed_price * item.count}} Raca</li>
                        <li><span>Số Lượng: </span>{{item.count}}</li>
                        <li><span>Tổng Tiền: </span>{{item.fixed_price}} Raca</li>
                        <li><a :href="item.urlMarket" target="_blank">Đến trang mua</a></li>
                      </ul>
                    </div>
                  </div>
                  <div class="col-md-3">
                    <div class="tt-item-text">
                      <ul>
                        <li><span>Total power:</span> {{item.totalPower}}</li>
                        <li><span>Rarity: </span>{{item.Rarity}}</li>
                        <li><span>Lucky:</span>{{item.Luck}}</li>
                        <li><span>Race:</span>{{item.Race}}</li>
                        <li><span>Stealth:</span>{{item.Stealth}}</li>
                        <li><span>Healthy:</span>{{item.Healthy}}</li>
                      </ul>
                    </div>
                  </div>
                  <div class="col-md-3">
                    <div class="tt-item-text">
                      <ul>
                        <li><span>Wisdom:</span>{{item.Wisdom}}</li>
                        <li><span>size: </span>{{item.Size}}</li>
                        <li><span>Courage:</span>{{item.Courage}}</li>
                        <li><span>Level:</span>{{item.Level}}</li>
                      </ul>
                    </div>
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
export default {
  data() {
    return {
      dataArr: null,
      dataArr1: null,
      tokenId: [],
      input2: '',
      loading: false,
      showHide: false,
      showCanQuetMoi: false,
      showNoNhanVat: false,
    }
  },
  methods: {
    onChangeTextSearch(event) {
      // console.log("The value is: ", event);
      this.nhanvat(event)
    },
    async nhanvat() {
      // console.log('Nhân vật ', data)
      // GET DATA nhân lực
      this.loading = true
      var dataNhanvat
      await axios.get('https://market-api.radiocaca.com/nft-sales?pageNo=1&pageSize=10000&sortBy=fixed_price&order=asc&name=&saleType&category=13&tokenType')
        .then(response => {
          dataNhanvat = response.data.list
        })
        // console.log('dataNhanvat', dataNhanvat)
      // this.dataArr = dataNhanvat
      // lấy tcoken_ Id
      var tokenId = []
      for (var i = 0; i < dataNhanvat.length; i++) {
        tokenId.push(dataNhanvat[i].token_id)
      }
      // console.log('token_id', tokenId)
      // Get TokenID to Link để lấy công thủ
      let congThuArr = []
      // var convertToLowerCase = data.toLowerCase()
      for (var j = 0; j < tokenId.length; j++) {
        // GET DATA CÔNG THỦ
        await axios.get('https://racawebsource.s3.us-east-2.amazonaws.com/metamon/metadata/normal/metamon_'+ tokenId[j])
        .then(response => {
          // console.log('congThuArr', response)
          let request = {
            'token_id': tokenId[j],
            'name': response.data.name,
            'image': response.data.image,
            'external_url': response.data.external_url,
            'description': response.data.description,
            'Courage': response.data.attributes[0].Courage,
            'Healthy': response.data.attributes[0].Healthy,
            'Level': response.data.attributes[0].Level,
            'Luck': response.data.attributes[0].Luck,
            'Race': response.data.attributes[0].Race,
            'Rarity': response.data.attributes[0].Rarity,
            'Size': response.data.attributes[0].Size,
            'Stealth': response.data.attributes[0].Stealth,
            'Wisdom': response.data.attributes[0].Wisdom,
          }
          congThuArr.push(request)
        })
      }
      // console.log('congThuArr', congThuArr)
      // GỘP 2 DATA LẠI
      let newData =[]
      for (let k = 0; k < dataNhanvat.length; k++){
        for (let q = 0; q < congThuArr.length; q++){
          if (dataNhanvat[k].token_id ===  congThuArr[q].token_id){
            let requestData = {
              'id': dataNhanvat[k].id,
              'name1':dataNhanvat[k].name1,
              'image_url1':dataNhanvat[k].image_url1,
              'count':dataNhanvat[k].count,
              'fixed_price':dataNhanvat[k].fixed_price,
              'highest_price':dataNhanvat[k].highest_price,
              'start_time':dataNhanvat[k].start_time,
              'end_time':dataNhanvat[k].end_time,
              'status':dataNhanvat[k].status,
              'sale_type':dataNhanvat[k].sale_type,
              'sale_address':dataNhanvat[k].sale_address,
              'token_id': congThuArr[q].token_id,
              'name': congThuArr[q].name,
              'image': congThuArr[q].image,
              'external_url': congThuArr[q].external_url,
              'description': congThuArr[q].description,
              'Courage': congThuArr[q].Courage,
              'Healthy': congThuArr[q].Healthy,
              'Level': congThuArr[q].Level,
              'Luck': congThuArr[q].Luck,
              'Race': congThuArr[q].Race,
              'Rarity': congThuArr[q].Rarity,
              'Size': congThuArr[q].Size,
              'Stealth': congThuArr[q].Stealth,
              'Wisdom': congThuArr[q].Wisdom,
              'totalPower': parseInt(congThuArr[q].Courage) + parseInt(congThuArr[q].Luck) + parseInt(congThuArr[q].Size) + parseInt(congThuArr[q].Stealth) + parseInt(congThuArr[q].Wisdom),
              'urlMarket': 'https://market.radiocaca.com/#/market-place/'+ dataNhanvat[k].id
            }
            newData.push(requestData)
          }
        }
      }
      // console.log('newData', newData)
      this.dataArr = newData.sort(function(a, b){
        return parseInt(b.totalPower) - parseInt(a.totalPower)
      })
      this.loading = false
      this.showHide = true
      this.showCanQuetMoi = true
    },
    handleSortLevel() {
      this.dataArr.sort(function(a, b){
        return parseInt(b.Level) - parseInt(a.Level)
      })
    },
    handleSortPower() {
      this.dataArr.sort(function(a, b){
        return parseInt(b.totalPower) - parseInt(a.totalPower)
      })
    }
  },
  computed: {
    queriedData () {
      return this.handleSortLevel()
    }
  },
  mounted () {
    this.nhanvat()
  }
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
    margin-bottom: 30px;
}
</style>
