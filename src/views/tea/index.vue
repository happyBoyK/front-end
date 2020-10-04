<template>
  <div>
    <Tabs :value="currTab">
      <TabPane
          v-for="(item,index) in tabList"
          :key="index"
          :label="item.cat.name"
          :name="'tea_' + index">
        <Row class="desk-list">
          <Card
              v-for = "(desk, index2) in item.list"
              :key="index2"
              span="11"
              :class="[desk.status ? (desk.status == 1 ? 'desk-orenge' : 'desk-red') : 'desk-green']"
              v-on:click.native="openModal(desk)"
              class="desk-item">
            <p slot="title" class="title">
              <span>{{ desk.name }}</span>
              <b class="status"></b>
            </p>
            <p class="item-call">
              <span>开始时间</span>
              <b>{{desk.start_time ? datetime(desk.start_time, 'DD HH:mm') : '未开始' }}</b>
            </p>
            <p class="item-call">
              <span>计费时长</span>
              <b>{{ desk.charging_type ? (desk.status == 2 ? desk.time_range : '未计时') : '不计时长'  }}</b>
            </p>
          </Card>
        </Row>
      </TabPane>
    </Tabs>

    <Modal v-model="modal11" @on-visible-change="modalVisibleChange"
           :fullscreen="true"
           footer-hide
           scrollable
           :title="modalTitle"
    >
      <div class="modal-body">
        <div class=" content modal-body-left">
          <Card v-if="modalData.status == 2">
            <Button v-on:click.native="openShopList" type="info" :style="{'margin-bottom': '20px'}">添加商品</Button>
            <Table border ref="selection" :columns="ShopColumns" :data="modalData.goodsList"></Table>
          </Card>
        </div>
        <div class="content modal-body-right">
          <table class="table m-b-none" v-if="modalData">
            <tbody>

            <tr>
              <td class="active">
                座位编号
              </td>
              <td> {{ modalData.name }} </td>
              <td class="active">
                上座时间
              </td>
              <td>{{ modalData.start_time ? datetime(modalData.start_time, "DD HH:mm") : '未开始' }}</td>
              <!-- <td>{{order.create_time|dateformat('Y-m-d H:i:s')}}</td> -->
              <td class="active">
                支付方式
              </td>
              <td>
                <Select v-model="pay_type" style="width:200px">
                  <Option v-for="(item, index) in pay_type_list" :value="item.id" :key="index">{{ item.pay_name }}</Option>
                </Select>
              </td>
            </tr>
            <tr>
              <td class="active">
                包间费
              </td>
              <td>
                <span class="text-danger">
                  ￥{{ modalData.basics_price }}
                </span>
              </td>
              <td class="active">
                时长
              </td>
              <td id="cat_duration">
                {{ modalData.charging_type ? (modalData.status == 2 ? modalData.time_range : '--') : '不计时长' }}
              </td>
              <td class="active">
                客源
              </td>
              <td>
                <Select v-model="guest_source" style="width:200px">
                  <Option v-for="(item, index) in guest_source_list" :value="item.id" :key="index">{{ item.name }}</Option>
                </Select>
              </td>
            </tr>
            <tr>
              <td class="active">
                超时时间
              </td>
              <td>
                <span class="text-danger">{{ modalData.status == 2 ?  modalData.out_date : '--'}}</span>
              </td>
              <td class="active">
                超时费单价
              </td>
              <td>
                <span class="text-danger">
                  {{ modalData.status == 2 ?  '￥' + (modalData.time_out_price * 2) : '--'}}
                </span>
              </td>
              <td class="active">
                超时费用
              </td>
              <td>
                <span class="text-danger">
                  {{ modalData.status == 2 ?  '￥' + modalData.total_time_out_price : '--'}}
                </span>
              </td>

            </tr>
            <tr>
              <td class="active">
                夜间服务时间
              </td>
              <td>
                <span class="text-danger">{{ modalData.status == 2 ?  modalData.other_out_date : '--'}}</span>
              </td>
              <td class="active">
                服务费单价
              </td>
              <td>
                <span class="text-danger">
                  {{ modalData.status == 2 ?  '￥' + (modalData.other_price * 2) : '--'}}
                </span>
              </td>
              <td class="active">
                夜间服务费用
              </td>
              <td>
                <span class="text-danger">
                  {{ modalData.status == 2 ?  '￥' + modalData.total_ohter_out_price : '--'}}
                </span>
              </td>

            </tr>
            <tr>
              <td class="active">
                商品总金额
              </td>
              <td>
                <span class="text-danger">
                    {{ modalData.status == 2 ? '￥' + modalData.goods_price : '--' }}
                </span>
              </td>
              <td class="active">
                代金券
              </td>
              <td><span class="text-danger">--</span></td>

              <td class="active">
                发票税金
              </td>
              <td><span
                  class="text-danger"></span> </td>
            </tr>
            <tr>
              <td class="active">
                原订单总金额
              </td>
              <td colspan="5"><span
                  class="text-danger olde_order_amount"></span>
                <span class="help-block m-b-none text-muted">
                  {{ modalData.status == 2 ? '￥' + modalData.total_price : '--' }}
                </span>
              </td>
            </tr>
            <tr>
              <td class="active">
                订单折扣或涨价
              </td>
              <td colspan="5"><span class="text-danger olde_order_amount">
                                        </span>
                <span class="help-block m-b-none text-muted">
                  <Input  number v-model="discount_price" placeholder="要给顾客便宜100元，则输入-100;要提高订单价格100元，则输入100" size="large" style="width: 400px;" />
                                        </span>
              </td>
            </tr>
            <tr>
              <td class="active">
                调整后订单总金额
              </td>
              <td colspan="5"><span class="text-danger"
                                    id="order_amount"></span>
                <span class="help-block m-b-none text-muted">
                {{ modalData.status == 2 ? '￥' + (modalData.total_price + (isNaN(Number(discount_price)) ? 0 : Number(discount_price))) : '--' }}
                </span></td>
            </tr>
            <tr>
              <td class="active">
                备注或调价原因
              </td>
              <td colspan="5">
                <Input  number v-model="order_remarks" placeholder="备注或者调价原因" size="large" style="width: 400px;" />
              </td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
      <footer class="modal-footer">
        <Button class="action-btn" v-if="modalData.status !== 2 && modalData.charging_type" :type="modalData.status == 1 ? 'info' : 'warning'" v-on:click.native="reserve">
          {{ modalData.status == 1 ? '取消预订' : '预定' }}
        </Button>
        <Button class="action-btn" v-if="modalData.status !== 2" type="success" v-on:click.native="startCharging">开始计费</Button>
        <Button class="action-btn" v-if="modalData.status == 2" v-on:click.native="orderSettlement" type="error">结账</Button>
      </footer>
    </Modal>

    <Modal v-model="shopModal" width="800" footer-hide scrollable :title="'选择商品'" style="background: #f5f5f5">
      <div class="goods-list-box" style="background: #f5f5f5">
        <Row :gutter="16" style="overflow: hidden;">
          <Col span="6" v-for="(item, index) in goodsList" :key="index" style="margin-bottom: 20px;">
            <Card v-on:click.native="selectGoods(item)">
              <p slot="title">
                {{ item.goods_name }}
              </p>
              <p>
                ￥{{item.price}}
              </p>
            </Card>
          </Col>

        </Row>
      </div>
    </Modal>

    <Modal v-model="orderCompleteModal" footer-hide :mask-closable="false" scrollable :title="'收款单，确认收到款后关闭!'" width="500">
      <List v-if="orderSettlementInfo" class="order-list">
        <ListItem>
          <b>座位号：</b> <span>{{ orderSettlementInfo.seat_name }}</span>
        </ListItem>
        <ListItem v-if="modalData.charging_type">
          <b>包间费：</b> <span>￥{{ orderSettlementInfo.room_price }}</span>
        </ListItem>
        <ListItem v-if="modalData.charging_type">
          <b>超时费：</b> <span>￥{{ orderSettlementInfo.room_out_price }}</span>
        </ListItem>
        <ListItem v-if="modalData.charging_type">
          <b>夜间服务费：</b> <span>￥{{ orderSettlementInfo.room_server_price }}</span>
        </ListItem>
        <ListItem>
          <b>商品总价：</b> <span>￥{{ orderSettlementInfo.goods_total_price }}</span>
        </ListItem>
        <ListItem>
          <b>结算总价：</b> <span>￥{{ orderSettlementInfo.total_price }}</span>
        </ListItem>
        <ListItem>
          <b>优惠价格：</b> <span>￥{{ orderSettlementInfo.discount_price }}</span>
        </ListItem>
        <ListItem>
          <b>结算：</b> <span style="font-size: 30px">￥{{ orderSettlementInfo.complete_price }}</span>
        </ListItem>
      </List>
      <footer class="ok-sk-footer">
        <Button type="success" v-on:click.native="orderComplete">确认已收到款</Button>
      </footer>

    </Modal>
  </div>

</template>

<script>
import util from '@/libs/util.js';
export default {
    name: 'index',
    data () {
        return {
            tabList: [],
            currTab: 'tea_1', // 当前的tab
            timerList: {}, // 定时器列表
            modal11: false,
            shopModal: false, // 商品列表显示
            modalTitle: '',
            modalData: {},
            modalTimer: null, // 弹窗定时器
            discount_price: 0, // 优惠价格或者加价价格
            order_remarks: '', // 订单备注
            orderCompleteModal: false, // 订单完成确认弹窗
            orderSettlementInfo: null,
            pay_type: null, // 支付方式
            pay_type_list: [], // 支付方式列表
            guest_source: null, // 客人来源
            guest_source_list: [], // 客人来源列表
            ShopColumns: [
                {
                    type: 'selection',
                    width: 60,
                    align: 'center'
                },
                {
                    title: '商品名',
                    key: 'goods_name'
                },
                {
                    title: '价格',
                    width: 80,
                    key: 'price',
                    render: (h, params) => {
                        return '￥' + params.row.price.toFixed(2);
                    }
                },
                {
                    title: '添加时间',
                    key: 'create_time',
                    render: (h, params) => {
                        return this.datetime(params.row.create_time, 'YYYY-MM-DD HH:mm');
                    }
                },
                {
                    title: '更新时间',
                    key: 'update_time',
                    render: (h, params) => {
                        if (!params.row.update_time) return '';
                        return this.datetime(params.row.update_time, 'YYYY-MM-DD HH:mm');
                    }
                },
                {
                    title: '操作',
                    key: 'action',
                    width: 80,
                    align: 'center',
                    render: (h, params) => {
                        const userName = localStorage.getItem('user');
                        if (userName !== 'admin') return h('div');
                        return h('div', [
                            h('Button', {
                                props: {
                                    type: 'error',
                                    size: 'small'
                                },
                                on: {
                                    click: () => {
                                        this.removeGoodsOrder(params.row.id, this.modalData.order_id);
                                    }
                                }
                            }, '删除')
                        ]);
                    }
                }
            ],
            goodsList: [] // 商品列表
        };
    },
    mounted () {
        this.GetCatList();
        this.GetPayListNet();
        this.GetGuestSource();
    },
    methods: {
        openModal (desk) {
            this.modal11 = true;
            this.modalTitle = desk.name;
            this.$Spin.show();
            this.getSeatInfo(desk.id);
        },
        /** 打开商品选择列表 **/
        openShopList () {
            this.shopModal = true;
            if (this.goodsList.length === 0) this.GetGoodsListNet(); // 获取商品列表
        },
        modalVisibleChange (e) {
            if (!e) clearInterval(this.modalTimer); // 关闭定时器
            if (e) this.modalData = {};
        },
        // 获取支付方式
        GetPayListNet () {
            this.post('admin/pay/payTypeList', {}, datas => {
                console.log('支付方式', datas);
                this.pay_type_list = datas;
            });
        },
        // 获取客人来源
        GetGuestSource () {
            this.post('admin/tea_open_order/guestSourceList', {}, datas => {
                this.guest_source_list = datas;
            });
        },
        // 获取商品列表
        GetGoodsListNet () {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/goodsList', {}, datas => {
                this.goodsList = datas;
            });
        },
        // 选择商品
        selectGoods (row) {
            this.$Spin.show();
            const params = {id: row.id, seat_id: this.modalData.id};
            util.post(this, 'admin/tea_open_order/selectGoods', params, datas => {
                this.shopModal = false;
                this.getSeatInfo(this.modalData.id);
            });
        },
        // 删除商品订单
        removeGoodsOrder (id, orderId) {
            this.$Modal.confirm({
                title: '删除订单',
                content: '确定要删除此订单吗？',
                onOk: () => {
                    this.removeGoodsOrderNet(id, orderId);
                }
            });
        },
        // 删除订单请求
        removeGoodsOrderNet (id, orderId) {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/removeGoodsOrder', {id, order_id: orderId}, datas => {
                this.getSeatInfo(this.modalData.id);
            });
        },
        /**
       * 获取座位详情
       * @param id
       */
        getSeatInfo (id) {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/seatinfo', { id }, (datas) => {
                this.modalData = datas;
                if (this.modalData.charging_type) if (this.modalData.charging_type) this.modalTimerStart();
                // 如果已结账则直接显示结账弹窗
                if (datas.order) {
                    this.orderSettlementInfo = datas.order;
                    this.orderCompleteModal = true;
                }
            });
        },
        reserve () {
            console.log('改座位数据', this.modalData);
            // 取消预订
            if (this.modalData.status === 1) {
                this.$Modal.confirm({
                    title: '取消预订',
                    content: '确认需要取消预订吗？',
                    onOk: () => {
                        this.reserveNet();
                    }
                });
            } else {
                this.reserveNet();
            }
        },

        reserveNet () {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/reserve', {id: this.modalData.id}, (datas) => {
                this.$Message.success(datas === 1 ? '预定成功！' : '取消成功!');
                this.modalData.status = datas;
                this.GetCatList();
            });
        },
        // 开始计费
        startCharging () {
            if (this.modalData.status === 2) return;
            this.$Modal.confirm({
                title: '确认计费',
                content: '确认开始计费吗？',
                onOk: () => {
                    this.startChargingNet();
                }
            });
        },
        // 订单结账
        orderSettlement () {
            this.$Modal.confirm({
                title: '确认结账',
                content: '确认结账吗？结账后订单将会锁定，无法更改！',
                onOk: () => {
                    // this.startChargingNet();
                    this.orderSettlementNet();
                }
            });
        },
        orderSettlementNet () {
            if (!this.pay_type) return this.$Message.warning('请先选择支付方式!');
            util.post(this, 'admin/tea_open_order/settlement',
                {guest_source: this.guest_source, pay_type: this.pay_type, id: this.modalData.id, discount_price: this.discount_price, order_remarks: this.order_remarks},
                datas => {
                    this.orderSettlementInfo = datas;
                    this.orderCompleteModal = true;
                });
        },
        orderComplete () {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/completeOrder', {id: this.modalData.order_id}, datas => {
                this.orderSettlementInfo = null;
                this.orderCompleteModal = false;
                this.modal11 = false;
                this.$Modal.success({
                    title: '订单已完成!',
                    onOk: () => {
                        this.GetCatList();
                    }
                });
            });
        },
        startChargingNet () {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/start', {id: this.modalData.id}, (datas) => {
                this.modalData.uuid = datas.uuid;
                this.modalData.start_time = datas.start_time;
                this.modalData.status = datas.status;
                this.modalData.order_id = datas.order_id;
                this.getSeatInfo(this.modalData.id);
                if (this.modalData.charging_type) this.modalTimerStart();
                this.GetCatList();
            });
        },
        /**
       * 计时器开始
       */
        modalTimerStart () {
            this.modalData.time_range = this.GetTimeRange(this.modalData.start_time);
            console.log('计时器开启', this.modalData.start_time, this.modalData.time_range);
            this.modalTimer = setInterval(() => {
                this.modalData.time_range = this.GetTimeRange(this.modalData.start_time);
            }, 60 * 1000);
        },
        GetCatList () {
            this.$Spin.show();
            util.post(this, 'admin/tea_open_order/index', {}, (datas) => {
                this.tabList = datas;
                for (const key in this.tabList) {
                    for (const item of this.tabList[key].list) {
                        if (item.charging_type && item.status === 2) this.SetMinTimer(item);
                    }
                }
            });
        },
        SetMinTimer (item) {
            console.log('当前开始时间', item.start_time);
            item.time_range = this.GetTimeRange(item.start_time);
            if (!this.timerList[item.id] && item.start_time) {
                this.timerList[item.id] = setInterval(() => {
                    item.time_range = this.GetTimeRange(item.start_time);
                }, 1000);
            }
        },
        GetTimeRange (start_time) {
            if (typeof start_time !== 'number') return '未开始计时';
            const end_time = Date.now();
            let ctime = end_time - start_time;
            ctime = Math.floor(ctime / 1000);
            const hour = Math.floor(ctime / (60 * 60)); // 计算小时
            const min = Math.floor((ctime - hour * 60 * 60) / 60); // 计算分钟
            return `${hour}小时${min}分`;
        }

    }

};
</script>

<style scoped>

.ok-sk-footer {
  width: 100%;
  height: auto;
  display: flex;
  justify-content: flex-end;
  position: absolute;
  bottom: 0;
  left: 0;
  padding: 20px;
}
.order-list {
  margin-bottom: 50px;
}
.goods-list-box {
  width: 100%;
  height: auto;
  box-sizing: border-box;
  padding: 20px;
}
.panel {
  border-radius: 2px;
}
.panel.panel-default {
  border-color: #eaeef1;
}
.panel.panel-default > .panel-heading,
.panel.panel-default > .panel-footer {
  border-color: #eaeef1;
}
.panel .list-group-item {
  border-color: #f3f5f7;
}
.panel.no-borders {
  border-width: 0;
}
.panel.no-borders .panel-heading,
.panel.no-borders .panel-footer {
  border-width: 0;
}
.panel .table td,
.panel .table th {
  padding: 8px 15px;
  border-top: 1px solid #eaeef1;
}
.panel .table thead > tr > th {
  border-bottom: 1px solid #eaeef1;
}
.panel .table-striped > tbody > tr:nth-child(odd) > td,
.panel .table-striped > tbody > tr:nth-child(odd) > th {
  background-color: #f7fafa;
}
.panel .table-striped > thead th {
  background-color: #f7fafa;
  border-right: 1px solid #eaeef1;
}
.panel .table-striped > thead th:last-child {
  border-right: none;
}
.panel-heading {
  border-radius: 2px 2px 0 0;
}
.panel-default .panel-heading {
  background-color: #f7fafa;
}
.panel-heading.no-border {
  margin: -1px -1px 0 -1px;
  border: none;
}
.panel-heading .nav {
  margin: -10px -15px;
}
.panel-heading .nav-tabs {
  margin: -11px -16px;
}
.panel-heading .nav-tabs.nav-justified {
  width: auto;
}
.panel-heading .nav-tabs > li > a {
  margin: 0;
  padding-top: 9px;
}
.panel-heading .list-group {
  background: transparent;
}
.panel-footer {
  border-color: #eaeef1;
  border-radius: 0 0 2px 2px;
  background-color: #f7fafa;
}
.panel-group .panel-heading + .panel-collapse .panel-body {
  border-top: 1px solid #eaedef;
}

.modal-body {
  width: 100%;
  height: calc(100% - 53px);
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
}
.modal-body .content {
  width: 100%;
}
.modal-body .modal-body-right table {
  width: 100%;
  height: auto;
  border: 1px solid #eeeeee;
  border-spacing: 0;
  border-collapse: collapse;
}
.modal-body .modal-body-right table tbody tr {
  width: 100%;
  height: 50px;
  border-bottom: 1px solid #eeeeee;
}
.modal-body .modal-body-right table tbody tr td {
  border-right: 1px solid #eeeeee;
  padding: 0 10px;
  min-width: 100px;
  font-size: 16px;
  color: #19be6b;
}
.modal-body .modal-body-right table tbody tr td.active {
  background: #f5f5f5;
  color: #788188;
}
.modal-body .modal-body-right table tbody tr td:last-child {
  border-right: 0px;
}
.modal-body .modal-body-right table tbody tr:last-child {
  border-bottom: 0px;
}
.modal-body .modal-body-right {

}
.modal-footer {
  position: absolute;
  bottom: 0px;
  left: 0px;
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
  padding: 10px 30px;
  border-top: 1px solid #cccccc;
  width: 100%;
}
.modal-footer .action-btn {
  margin-right: 20px;
}
.desk-list{
  width: 100%;
  display:flex;
  display: -webkit-flex; /* Safari */
  flex-wrap: wrap;
  justify-content: space-between;
}

.ivu-row:after, .ivu-row:before {
  display: none;
}
.desk-item {
  width: 300px;
  height: auto;
  flex:0 0 30%;
  margin-bottom: 30px;
}

.desk-item .title {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
.desk-item b.status {
  display: block;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  box-shadow: 0 1px 6px rgba(0,0,0,.2);
}
.desk-item:hover {
  cursor: pointer;
}
.desk-item.desk-green b.status {
  background: #19be6b;
}
.desk-item.desk-orenge b.status {
  background: #f90;
}
.desk-item.desk-red b.status {
  background: #ed4014;
}
.desk-item::before {
  display: none;
}

.item-call {
  border-bottom: 1px solid #eeeeee;
}


.item-call span {
  display: inline-block;
  width: 100px;
  height: 100%;
  padding: 10px;
  background: #f5f5f5;
  text-align: right;
}
</style>
