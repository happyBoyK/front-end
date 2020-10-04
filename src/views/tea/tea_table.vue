<template>
  <div>
    <div class="table-content">
      <Table border ref="selection" :columns="columns" :data="list">
        <template slot-scope="{ row, index }" slot="status">
          <Tag :color="row.status ? 'success' : 'error'">{{ row.status ? '完成' : '付款' }}</Tag>
        </template>
      </Table>

    </div>

    <Page class="page-bar" :total="count"
          show-sizer
          @on-change="PageChange"
          @on-page-size-change="PageSizeChange"
          :page-size="pageSize"
          :current="page" />
  </div>
</template>

<script>
import util from '@/libs/util.js';
export default {
    name: 'tea_table',
    data () {
        return {
            columns: [
                {
                    type: 'selection',
                    width: 60,
                    align: 'center',
                    title: 'id',
                    key: 'id'
                },
                {
                    title: '座位名称',
                    key: 'seat_name'
                },
                {
                    title: '超时费',
                    key: 'room_out_price'
                },
                {
                    title: '服务费',
                    key: 'room_server_price'
                },
                {
                    title: '商品总价',
                    key: 'goods_total_price'
                },

                {
                    title: '优惠、提价',
                    key: 'discount_price'
                },
                {
                    title: '实际结算',
                    key: 'complete_price'
                },
                {
                    title: '结算时间',
                    key: 'order_end_time',
                    width: '200',
                    render: (h, params) => {
                        return this.datetime(params.row.order_end_time, 'YYYY-MM-DD HH:mm');
                    }
                },
                {
                    title: '支付方式',
                    key: 'pay_name'
                },
                {
                    title: '订单状态',
                    key: 'status',
                    slot: 'status'
                },
                {
                    title: '结算用户',
                    key: 'nickname'
                }
            ],
            list: [],
            page: 0,
            pageSize: 20,
            count: 0
        };
    },
    mounted () {
        this.orderLisCountNet();
        this.page = this.page + 1;
    },
    watch: {
        page (val) {
            console.log('页码变化', val);
            this.GetListNet();
        },
        pageSize (val) {
            this.GetListNet();
        }
    },
    methods: {
        orderLisCountNet () {
            util.post(this, 'admin/tea_open_order/orderLisCount', {}, datas => {
                this.count = datas;
            });
        },
        GetListNet () {
            util.post(this, 'admin/tea_open_order/orderList?page=' + this.page + '&pageSize=' + this.pageSize, {}, datas => {
                this.list = datas;
            });
        },
        PageChange (page) {
            this.page = page;
        },
        PageSizeChange (pageSize) {
            this.pageSize = pageSize;
        }
    }
};
</script>

<style scoped>
.page-bar {
 width: 100%;
  display: flex;
  justify-content: flex-end;
  box-sizing: border-box;
  padding: 20px 0;
  padding-right: 20px;
}

</style>
