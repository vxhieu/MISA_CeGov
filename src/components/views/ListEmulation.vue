<template>
  <div class="tableData">
    <div class="tableData--top">
      <table>
        <thead>
          <tr>
            <th>
              <div><input class="select--value" type="checkbox" @click="changeChecked"/>
                <div v-if="isSelected" class="select--multi" @click="deleteChecked"></div>
              </div>
             
            </th>
            <th  class="reward--sticky" style="min-width: 310px; max-width: 310px">
              <div >Tên danh hiệu thi đua</div>
            </th>
            <th style="width: 160px; max-width: 160px; min-width: 160px">
              <div>Mã danh hiệu</div>
            </th>
            <th style="width: 200px; max-width: 200px; min-width: 200px">
              <div>Đối tượng khen thưởng</div>
            </th>
            <th style="width: 200px; max-width: 200px; min-width: 200px">
              <div>Cấp khen thưởng</div>
            </th>
            <th style="width: 200px; max-width: 200px; min-width: 200px">
              <div>Loại phong trào</div>
            </th>
            <th style="width: 180px; max-width: 180px; min-width: 180px">
              <div>Trạng thái</div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, key) in this.listValue"
            :key="key"
            @dblclick="logData(item)"
            @click="changeBackground(item)"
          >
            <td>
              <div class="clickSelect">
                <input
                  class="select--value"
                  @click="selectValue(item.rewardID)"
                  type="checkbox"
                  v-model="this.listSelect"
                  :value=item.rewardID
                 
                />
              </div>
            </td>
            <td class="reward--sticky"><div class="reward--sticky__wrap">{{ item.rewardName }}</div></td>
            <td>{{ item.rewardCode }}</td>
            <td>
              {{
                item.rewardObject == 2
                  ? "Tập thể;Cá Nhân"
                  : item.rewardObject == 0
                  ? "Cá Nhân"
                  : "Tập thể"
              }}
            </td>
            <td>{{ this.enumsEmulation[`${item.levelID}`] }}</td>
            <td>
              {{
                item.rewardType == 2
                  ? "Thường xuyên;Theo đợt"
                  : item.levelReward == 0
                  ? "Thường xuyên"
                  : "Theo đợt"
              }}
            </td>
            <td>
              <span v-if="( item.rewardStatus ==0 )" class="actived"></span>
              <span v-if="( item.rewardStatus == 1)" class="not-active"></span>
              {{ item.rewardStatus == 0 ? "Sử dụng" : "Ngừng sử dụng" }}
            </td>
            <div class="manageOption">
              <div class="manageOption--select">
                <div class="manageOption--select__edit">
                  <div class="icon bg"></div>
                </div>
                <div
                  class="manageOption--select__more"
                  @click="showMoreForm(event)"
                >
                  <div class="icon bg"></div>
                </div>
              </div>
              <div
                class="more--form"
                v-if="this.active"
                @mouseleave="changeActive"
              >
                <div
                  class="more--form__use"
                  :class="{ disabled: item.rewardStatus == 0 }"
                >
                  Sử dụng
                </div>
                <div
                  class="more--form__stop"
                  :class="{ disabled: item.rewardStatus == 1 }"
                >
                  Ngừng sử dụng
                </div>
                <div class="more--form__delete" @click="getDataRow(item)">
                  Xoá
                </div>
              </div>
            </div>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="tableData--paging">
      <div class="tableData--paging__left">
        Tổng số <span>{{ Object.keys(listValue).length }}</span> bản ghi
      </div>
      <div class="tableData--paging__right">
        <div class="paging--right__index">
          <div class="index--para" >Số bản ghi/trang</div>
          <div class="index--nunmber">
            <Dropdown
                  class="select drop-down"
                  v-model="selectedPaging"
                  :options="rewardPaging"
                  optionLabel="name"
                  placeholder="10"
                />
          </div>
          <div class="index--para"><span></span> bản ghi</div>
          <div class="index--nav">
            <div class="index--nav__prev bg"></div>
            <div class="index--nav__move bg"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Dropdown from "primevue/dropdown";
import axios from "axios";
import $ from "jquery";
//  Tập thể:0 Cá nhân:1 Tập thể và cá nhân 2
//  Theo đợt 0 Thường xuyên 1; Thường xuyên và theo đợt

export default {
  props: {
   
  },
  components: {
    Dropdown,
  },
  data() {
    return {
      listValue: {},
      listSelect: [],
      listID:[],
      isSelected:false,
      active: false,
      isChecked:false,
      enumsEmulation:{
        1: "Cấp Nhà nước",
        2: "Cấp Tỉnh/tương đương",
        3: "Cấp Huyện/tương đương",
        4: "Cấp Xã/tương đương"
      }, selectedPaging: null,
      rewardPaging: [
        { name: "10", code: "NY" },
        { name: "20", code: "RM" },
        { name: "50", code: "RM" },
        { name: "100", code: "LDN" },
      ],
    };
  },
  created() {
    this.getAllData();
  },
  mounted() {
    this.scrollTable(), this.selectValue();
   

    /* eslint-env jquery */
  },
  methods: {
    /**
     * Author :VxHieu
     * 13/11/2022
     */
    //   Thay đổi màu background của hàng khi click chuột
    //  Thay đổi màu click xung quanh thành màu trắng
    changeBackground() {
      try {
        var ele;
        $("tbody tr").click(function () {
          var array = $(this).siblings();
          ele = $(this);
          for (var i = 0; i < array.length; i++) {
            if (
              $(array[i]).children().css("background-color") ==
              "rgb(224, 235, 255)"
            ) {
              $(array[i]).children().css("background-color", "");
            }
          }
          $(this).children().css("background-color", " #e0ebff");
        });

        $("body").click(function (event) {
          if (!$(event.target).is("td")) {
            ele.children().css("background-color", "");
          }
        });
      } catch (error) {
        console.log(error);
      }
    },

    // thanh cuộn scroll sang trái
    scrollTable() {
      try {
        $("table").on("scroll", function () {
          $("table > *").width($("table").width() + $("table").scrollLeft());
        });
      } catch (error) {
        console.log(error);
      }
    },
    logData(item) {
      try {
        this.$emit("valueReward", item);
      } catch (error) {
        console.log(error);
      }
    },
    /**
     * Author :VxHieu
     * 14/11/2022
     */

    // trả dữ liệu của hàng khi click chuột vào checkbox
    selectValue(item) {
      try {
        if (item) {
          this.isSelected=true;
          
          if (this.listSelect.length == 0) {
            this.listSelect.push(item);
            
          } else if (jQuery.inArray(item, this.listSelect) == -1) {
            this.listSelect.indexOf(item);
            this.listSelect.push(item);
          } else {
            console.log( this.listSelect);
            this.listSelect.length == 1 ? this.isSelected=false : "";
            this.listSelect.splice(this.listSelect.indexOf(item),1);
          }
        }
        this.$emit("valueSelect", this.listSelect);
      } catch (error) {
        console.log(error);
      }
    },
    deleteChecked(){
      this.isSelected=false;
      this.listSelect=[];
      this.$emit("valueSelect", this.listSelect);
    },
    /**
     * Author :VxHieu
     * 23/11/2022
     */
    //Call Api lấy danh sách tất cả các danh hiệu thi đua
    getAllData() {
      axios
        .get("http://localhost:5194/api/Rewards")
        .then((response) => {
          this.listValue = response.data;
          
        })
        .catch((e) => {
          console.log(e);
        });
    },
    /**
     * Author :VxHieu
     * 24/11/2022
     */
    // nhấn vào icon thêm nữa để hiện ra các lựa chọn sử dụng, ngừng sử dụng, xoá
    showMoreForm(e) {
      console.log(e);
      this.active = true;
    },
    changeActive() {
      this.active = false;
    },

    /**
     * Author:VxHieu
     * 27/11/2022
     */
    getDataRow(item) {
      try {
        this.$emit("deleteValue", item);
      } catch (error) {
        console.log(error);
      }
    },
     /**
     * Author:VxHieu
     * 4/12/2022
     */
    //kiểm tra sự kiện click vào checkbox
    changeChecked(){
      if(this.listID.length==0){
        this.listValue.forEach(element => {
        this.listID.push(element.rewardID);
      });
      }
      this.isChecked=!this.isChecked;
      if(!this.isChecked){
        this.listSelect.splice(0,this.listSelect.length);
      }else{
        this.listSelect=[];
        if(this.listSelect.length==0){
        this.listSelect.push(...this.listID);
        }
      }
      this.$emit("valueSelect", this.listSelect);
     
    },
  
  },
};
</script>
<style scoped>
@import url("../../css/layout/form_detail.css");

</style>
