/* * @Author: casablca * *@Date: 2018-05-23 08:55:48 * * @Last Modified by: mikey.zhaopeng * @Last Modified time: 2018-05-23
12:13:02 08:56:13 */

<template>
	
		<div class="panel-body" style="padding-bottom:0px;">
			<div id="panel_search" class="panel panel-primary">
				<div class="panel-heading">查询条件</div>
				<div class="panel-body table-parent-panel">
					<form id="formSearch" class="form-horizontal">
						<div class="form-group" style="margin-top:15px">
							<label class="control-label col-sm-1" for="txt_search_departmentname">姓名</label>
							<div class="col-md-2">
								<select class="form-control" v-model="selected_user">
									<option v-for='option in options_user' v-bind:value="option.value">
										{{option.text}}
									</option>
								</select>
							</div>
							<label class="control-label col-sm-1" for="txt_search_departmentname">组名</label>
							<div class="col-md-2">
								<select class="form-control" v-model="selected_group">
									<option v-for='option in options_group' v-bind:value="option.value">
										{{option.text}}
									</option>
								</select>
							</div>
							<label class="control-label col-sm-1" for="txt_search_statu">日期</label>
							<div class="col-md-2">
								<div class="form-group">
									<div class="input-group date form_date col-md-12" data-date="" data-date-format="yyyy MM dd  " data-link-field="dtp_input2"
									    data-link-format="yyyy-mm-dd">
										<input id="datetimepicker" class="form-control" size="16" type="text" value="">
										<span class="input-group-addon">
											<span class="glyphicon glyphicon-calendar"></span>
										</span>
									</div>
									<input type="hidden" id="dtp_input2" value="" />
									<br/>
								</div>
							</div>
							<div class="col-md-2" style="text-align:left;">
								<!-- <button type="submit" id="btn_search" type="button" style="margin-left:50px" id="btn_query" class="btn btn-primary">查询</button> -->
								<button type="button" id="btn_search" style="margin-left:50px" v-on:click="summit()" class="btn btn-primary">查询</button>
							</div>
						</div>
					</form>
				</div>
			</div>

		</div>
	
</template>

<script>
import bus from "../assets/eventBus";
// import vue from 'vue'
export default {
  //父组件传入的参数
  props: ["searchResult"],
  // props:{
  // 	searchResult:{
  // 		type:,
  // 		required:true //参数一定要传入
  // 	}
  // },
  data: function() {
    return {
      selected_user: 1,
      options_user: [],
      selected_group: 2,
      options_group: [],
      selected_date: null,
      dict_users: {},
      searchResult: null
    };
  },
  watch: {
    /*
                            监听：
                                选中的用户，
                                选中的群组
    
                        */
    selected_date: function(new_val, old_val) {
      // alert(new_val);
    },
    selected_user: function(new_val, old_val) {},
    selected_group: function(new_val, old_val) {
      var self = this;
      //self.options_user=dict_users[new_val];
      this.options_user = [];
      this.options_user.push({
        text: "不选择",
        value: 0
      });
      var options_user = this.options_user;
      var dict_users = this.dict_users;
      $.map(dict_users[new_val], function(obj_user) {
        options_user.push({
          text: obj_user.username,
          value: obj_user.uid
        });
      });
      this.options_user = options_user;
      this.dict_users = dict_users;
      //select_options.options_user.push({text:obj.name,value:obj.did})
    }
  },
  methods: {
    dateDefind: function() {
      var d, s;
      var myself = this;
      d = new Date();
      s = d.getFullYear() + "-"; //取年份
      s = s + (d.getMonth() + 1) + "-"; //取月份
      s += d.getDate() + " "; //取日期
      // s += d.getHours() + ":";    //取小时
      // s += d.getMinutes() + ":";  //取分
      // s += d.getSeconds();     //取秒
      myself.time = s;
      //初始化
      $("#datetimepicker").datetimepicker({
        // startDate: s,
        minView: "month", //选择日期后，不会再跳转去选择时分秒
        language: "zh-CN",
        format: "yyyy-mm-dd",
        todayBtn: 1,
        autoclose: 1
      });
      //当选择器隐藏时，讲选择框只赋值给data里面的time
      $("#datetimepicker")
        .datetimepicker()
        .on("hide", function(ev) {
          var value = $("#datetimepicker").val();
          myself.selected_date = value;
          // myself.modifiedDate(value);
          //注意此处的this已经不是外面的作用域了，此处的this为datatimepicker
          // this.selected_date = value;
        });
    },
    modifiedDate: function(date_val) {
      // this.selected_date=date_val;
    },
    dataInit: function() {
      var myself = this;
      this.selected = 3;
      this.selected_group = 2;
    },
    testsummit: function() {
      bus.$emit("on-message", "search组件的消息");
    },
    summit: function() {
      var search_url = "http://127.0.0.1:8000/duty/schedulelist/";
      //注意若想让vue中的方法访问data，需要使用this，最好通过self=this的方式
      var myself = this;
      /*
					此处修改如下：
						1、将查询的条件交给兄弟组件content
						2、兄弟组件通过bootstrap-table 的load方法进行加载，
						3、不在本组件内进行ajax处理

				*/

      // 使用下面的es6的语法实现
      //   var select_group_name = myself.options_group.map(function(temp) {
      //     if (temp.value == myself.selected_group) {
      //       return temp.text;
      //     }
      //   });

      var select_group_name = myself.options_group.map(temp => {
        if (temp.value == myself.selected_group) {
          return temp.text;
        }
      });
      select_group_name = select_group_name[0];

      var group_temp = {
        text: select_group_name,
        value: myself.selected_group
      };


      //注意这个obj是要请求到后台的
      var search_temp = {
        user_id: myself.selected_user,
        users_id:[myself.selected_user],
        group: group_temp,
        group_id: myself.selected_group,
        groups_id: [myself.selected_group],
        // group_name: select_group_name,
        selected_date: myself.selected_date
      };
      //每次提交时，先清空content组件的data
      bus.$emit("on-clearData");

      bus.$emit("on-loadTable", search_url, search_temp);

      // 以下注释掉，不再使用，交给content兄弟组件

      // $.ajax({
      // 	url: search_url,
      // 	type: 'GET',
      // 	data: {
      // 		user_id: myself.selected_user,
      // 		group_id: myself.selected_group,
      // 		selected_date: myself.selected_date
      // 	},
      // 	async: false,
      // 	success: function (res) {
      // 		// alert(res);
      // 		myself.searchResult=res;
      // 		bus.$emit('on-searchresult',res);
      // 	}
      // })
    },
    getSchedulelist: function() {},
    //获取群组和群组对应的人员
    getgroupuser() {
      // var self = this;
      var data_get = null;
      var myself = this;
      // var temp=this;
      var get_groupAnduser_url = "http://127.0.0.1:8000/duty/grouplist/";
      var post_data = {
        department_id: 1
      };
      var dict_users = {};
      $.ajax({
        type: "GET",
        url: get_groupAnduser_url,
        dataType: "json",
        data: {
          post_data
        },
        async: false,
        success: function(data) {
          /*
                          1、遍历一级，存入group中
                          2、
                        */
          data_get = data;
        }
      }).then(function(data) {});
      // dict_users = {};
      var options_group = this.options_group;
      var dict_users = this.dict_users;
      $.map(data_get, function(obj) {
        options_group.push({
          text: obj.name,
          value: obj.did
        });
        if (obj.uid.length > 1) {
          dict_users[obj.did] = obj.uid;
        }
      });
      this.options_group = options_group;
      this.dict_users = dict_users;
    }
  },

  mounted: function() {
    this.dateDefind();
    this.getgroupuser();
  }
};
</script>
<style>
</style>