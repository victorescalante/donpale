<template>
  <div>
    <el-row>
      <el-col :span="8">
        <div class="block">
          <el-date-picker
            v-model="date"
            type="daterange"
            align="right"
            start-placeholder="Fecha de inicio"
            end-placeholder="Fecha final"
            :picker-options="pickerOptions"
          >
          </el-date-picker>
        </div>
      </el-col>
      <el-col :span="2">
        <el-button @click="getReportBy">Aplicar filtro</el-button>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="12">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>Productos más vendidos</span>
          </div>
          <div class="body-card table-products-sale">
            <el-table
              :data="showData"
              stripe
              class="style_table_info">
              <el-table-column
                prop="sku"
                label="SKU"
                width="150"
              >
              </el-table-column>
              <el-table-column
                prop="name"
                label="Nombre">
              </el-table-column>
              <el-table-column
                prop="seller"
                label="Vendedor"
                width="150"
              >
              </el-table-column>
              <el-table-column
                prop="sum"
                label="Pzas Vendidas"
                width="150"
              >
              </el-table-column>
            </el-table>
            <el-pagination
              class="style-pagination"
              layout="prev, pager, next"
              :page-size="total_for_page"
              @current-change="handleCurrentChange"
              :total="sort_collection.length">
            </el-pagination>
          </div>
        </el-card>
      </el-col>
      <el-col :span="12" >
        <div style="text-align: center">
          <h2>Total de ventas: {{ total_sales }}</h2>
        </div>
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>Lista de productos vendidos</span>
          </div>
          <div class="body-card">
            <el-collapse accordion>
              <div v-for="(seller, index) in group_by_seller" >
                <el-collapse-item class="item-style-header" :title="index" :name="index">
                  <el-table
                    :data="seller"
                    style="width: 100%">
                    <el-table-column
                      prop="sku"
                      label="SKU"
                      width="150"
                    >
                    </el-table-column>
                    <el-table-column
                      prop="name"
                      label="Nombre">
                    </el-table-column>
                    <el-table-column
                      prop="sum"
                      label="Vendidos"
                      width="100"
                    >
                    </el-table-column>
                  </el-table>
                </el-collapse-item>
              </div>
            </el-collapse>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import firebase from 'firebase/app'
  import 'firebase/firestore'
  let _ = require('lodash');
  let dayjs = require('dayjs');
  dayjs.locale('mx');
  import { Polar, Pie } from 'laue'

  let instance_firebase = null;
  if (!firebase.apps.length){
    instance_firebase = firebase.initializeApp({
      apiKey: 'AIzaSyDTiguxvO7DDnGNc93aRKPnbHNAqr5xp2w',
      authDomain: 'donpale-a045c.firebaseapp.com',
      databaseURL: 'https://donpale-a045c.firebaseio.com',
      projectId: 'donpale-a045c',
      storageBucket: 'donpale-a045c.appspot.com',
      messagingSenderId: '624280255695',
      appId: '1:624280255695:web:e5823f32f79e5035464bba'
    });

  }else{
    instance_firebase = firebase.app()
  }

  let db = instance_firebase.firestore();

  const { Timestamp, GeoPoint } = firebase.firestore;
  export { Timestamp, GeoPoint }

  export default {
    name: "report",
    data(){
      return{
        sort_collection: [],
        sort_pagination_collection: [],
        current_page: 0,
        total_sales: 0,
        total_for_page: 10,
        merge_data: [],
        group_by_seller: [],
        date: '',
        pickerOptions: {
          shortcuts: [{
            text: 'Hoy',
            onClick(picker) {
              picker.$emit('pick', [dayjs().startOf('day').format(), dayjs().format()]);
            }
          }, {
            text: 'Esta semana',
            onClick(picker) {
              picker.$emit('pick', [dayjs().startOf('week').format(), dayjs().format()]);
            }
          }, {
            text: 'Este mes',
            onClick(picker) {
              picker.$emit('pick', [dayjs().startOf('month').format(), dayjs().format()]);
            }
          }]
        }
      }
    },

    computed:{
      showData(){
        return this.sort_pagination_collection[this.current_page];
      }
    },

    mounted: function () {
      this.$nextTick(function () {
        this.date = [dayjs().startOf('day').format(),dayjs().format()];
        this.getReportBy();
      })
    },

    methods: {

      handleCurrentChange(val){
        this.current_page = val-1;
      },

      async getReportBy() {
        let sku_sales = await this.getSaleSKUs(this.date);
        let products = await this.getProductsBySKU(sku_sales[0]);
        let merge = [];
        _.forEach(sku_sales[0], function (value) {
          let index = _.findIndex(products, function(o) { return o.sku === value.sku; });
          merge.push(_.merge(value, products[index]));
        });
        console.log('sku_sales ', sku_sales)
        this.total_sales = sku_sales[1];
        this.merge_data = merge;
        this.group_by_seller = _.groupBy(merge, 'seller');
        this.sort_collection = _.orderBy(this.merge_data, ['sum'],['desc']);
        this.sort_pagination_collection = _.chunk(this.sort_collection, this.total_for_page)
      },

      getSaleSKUs: async function (date_range) {

        let products = [];
        let sales_count = [];

        let sales = db.collection("sales")
          .where('created_at', '>=', new Date(date_range[0]))
          .where('created_at', '<', new Date(dayjs(date_range[1]).endOf('day').format()));

        let vals = await sales.get().then((snapshot) => {

          /**
           * Obtener todas las ventas seleccionadas
           */
          snapshot.forEach((sale) => {
            let current_sale = sale.data();
            for (let i = 0; i < current_sale.products.length; i++) {
              if (!products.find(element => element === current_sale.products[i].sku)) {
                console.log('current_sale ', current_sale)
                products.push({
                  'sku': current_sale.products[i].sku,
                  'count': current_sale.products[i].units
                });
              }
            }
            sales_count.push({
              'total': current_sale.total
            })
          });

          /**
           * Agrupar los productos vendidos (Son los productos de la venta)
           */
          let group = _.groupBy(products, 'sku');
          let products_by_sku = [];
          _.forEach(group, function (array, key) {
            let sum_products = _.reduce(array, function (sum, n) {
              return sum + parseInt(n.count)
            }, 0);
            products_by_sku.push({'sku': key, 'sum': sum_products})
          });

          let total_sales = _.reduce(sales_count, function (sum, number) {
            console.log('number ', number)
            return sum + parseInt(number.total)
          }, 0);

          return [products_by_sku, total_sales]

        });

        return vals;

      },

      async getProductsBySKU(products_by_sku) {

        let x = [];
        let products_in_sale = [];

        /**
         * Para realizar consultas en firestore, lo hacemos obteniendo chunks de 10,
         * Es ul número máximo que acepta la consulta IN en Firebase
         * @type {Array}
         */
        let chunks_product = _.chunk(products_by_sku, 10);

        _.forEach(chunks_product, async function (value, key) {

          let only_sku = _.reduce(value, function (result, value) {

            result.push(value.sku);
            return result
          }, []);

          /**
           * Por cada Chunk de datos, obtenemos pequeñas colecciones para hacer relación entre
           * los productos.
           * @type {Promise<firebase.firestore.QuerySnapshot<firebase.firestore.DocumentData>>}
           */

          x.push(
            db.collection('products').where('sku', 'in', only_sku).get().then(
              async function (querySnap) {
                await querySnap.forEach(function (doc) {
                  let object = doc.data();
                  products_in_sale.push({
                    'sku': object.sku,
                    'name': object.name,
                    'seller': object.seller
                  })
                });
              }.bind(key))
          );

        });

        await Promise.all(await x).then((result) => {
          console.log("Consultas realizads");
        });

        return products_in_sale
      }

    },

  }
</script>

<style>

  .style_table_info{
    width: 100%;
  }
  .style_table_info .el-table__body-wrapper{
    text-transform: uppercase;
  }
  .style-pagination{
    padding: 15px;
    text-align: center;
  }

  .item-style-header{
    text-transform: uppercase;

  }

  .el-row{
    margin-bottom: 20px;
  }

  .el-col{
    padding: 15px;
  }

  .table-products-sale table{
    font-size: 12px;
  }
</style>
