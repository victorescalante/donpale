<template>
  <div v-loading.fullscreen.lock="findActive">
    <el-row :gutter="24">
      <el-col :span="2">
        <nuxt-link to="/">
          <el-button type="primary" icon="el-icon-s-home"></el-button>
        </nuxt-link>
      </el-col>
      <el-col :span="10" class="search_box">
        <form v-on:submit="onSubmit">
          <el-row>
            <el-col>
              <el-input placeholder="Buscar productos"
                        v-model="search" ref="search" class="search_input">
              </el-input>
            </el-col>
          </el-row>
        </form>
      </el-col>
      <el-col :span="8" style="padding: 0px 10px">
        <el-button type="primary" @click.stop="findProductsByName()" icon="el-icon-search">Buscar</el-button>
        <el-button type="secondary" @click="clearSearch()" v-if="search.length > 0">Limpiar Busqueda</el-button>
      </el-col>
      <el-col :span="4" style="text-align: right">
        <el-button type="success" @click.stop="finishSale()" v-if="this.productsInSale.length > 0">Realizar cobro</el-button>
      </el-col>
    </el-row>
    <el-row :gutter="24">
      <el-row class="space-box">
        <el-col :span="18">
          <div class="my_keyboard_two">
            <div class="line_keyboard">
              <el-button @click="addLetter('q')">q</el-button>
              <el-button @click="addLetter('w')">w</el-button>
              <el-button @click="addLetter('e')">e</el-button>
              <el-button @click="addLetter('r')">r</el-button>
              <el-button @click="addLetter('t')">t</el-button>
              <el-button @click="addLetter('y')">y</el-button>
              <el-button @click="addLetter('u')">u</el-button>
              <el-button @click="addLetter('i')">i</el-button>
              <el-button @click="addLetter('o')">o</el-button>
              <el-button @click="addLetter('p')">p</el-button>
            </div>
            <div class="line_keyboard_b_two">
              <el-button @click="addLetter('a')">a</el-button>
              <el-button @click="addLetter('s')">s</el-button>
              <el-button @click="addLetter('d')">d</el-button>
              <el-button @click="addLetter('f')">f</el-button>
              <el-button @click="addLetter('g')">g</el-button>
              <el-button @click="addLetter('h')">h</el-button>
              <el-button @click="addLetter('j')">j</el-button>
              <el-button @click="addLetter('k')">k</el-button>
              <el-button @click="addLetter('l')">l</el-button>
              <el-button @click="addLetter('ñ')">ñ</el-button>
            </div>
            <div class="line_keyboard_c_two">
              <el-button @click="addLetter('z')">z</el-button>
              <el-button @click="addLetter('x')">x</el-button>
              <el-button @click="addLetter('c')">c</el-button>
              <el-button @click="addLetter('v')">v</el-button>
              <el-button @click="addLetter('b')">b</el-button>
              <el-button @click="addLetter('n')">n</el-button>
              <el-button @click="addLetter('m')">m</el-button>
            </div>
          </div>
        </el-col>
        <el-col :span="6" style="text-align: right">
          <div class="total_container">
            <h2>Total de Venta</h2>
            <div class="total_sale">
              <p><span>$</span> {{ sumSale }}</p>
            </div>
          </div>
        </el-col>
      </el-row>
      <el-col :span="24">
        <div class="table_sale">
          <el-row :gutter="12" class="products_detail">
            <el-col :span="8">Nombre</el-col>
            <el-col :span="5">Unidades</el-col>
            <el-col :span="3">Precio</el-col>
            <el-col :span="4">Subtotal</el-col>
          </el-row>
          <el-row :gutter="12" v-for="product in productsInSale" :key="product.id" class="products_table">
            <el-col :span="8">{{ product.name }}</el-col>
            <el-col :span="5"><el-input-number @change="handleChange"  :min="1" v-model="product.units"></el-input-number></el-col>
            <el-col :span="3">$ {{ product.price }}</el-col>
            <el-col :span="4">$ {{ product.units * product.price }}</el-col>
            <el-col :span="4"><el-button type="danger" href="#" @click.prevent="DeleteOfSale(product.id)">Eliminar</el-button></el-col>
          </el-row>
        </div>
      </el-col>
    </el-row>

    <el-dialog
      title="Busqueda de productos"
      :visible.sync="dialogVisible"
      fullscreen>
      <div>
        <el-table
          stripe
          :data="combineFindAll"
          style="width: 100%"
          max-height="450">
          <el-table-column
            style="font-weight: bold"
            prop="name"
            label="Nombre"
            width="350"
          >
          </el-table-column>
          <el-table-column
            prop="sku"
            label="Código">
          </el-table-column>
          <el-table-column
            prop="price"
            label="Precio"
          >
          </el-table-column>
          <el-table-column
            fixed="right"
            label="">
            <template slot-scope="scope">
              <el-button
                @click.native.prevent="addProductButton(scope.$index, combineFindAll)"
                type="text"
                size="small">
                Agregar
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-dialog>

    <el-dialog
      title="Resumen de venta"
      :visible.sync="dialogVisibleExit"
      fullscreen
      :before-close="handleClose">
      <div>
        <el-row>
          <el-col :span="5">
            <h3>Total de Compra</h3>
            <div class="total_box">$ {{ sumSale }}</div>
            <h3>Pago del Cliente</h3>
            <div class="total_box">$ {{ clientPaymentToInt }}</div>
            <h3>Cambio</h3>
            <div class="total_box" style="background: lightseagreen">$ {{ client_payment - sumSale }}</div>
          </el-col>
          <el-col :span="7" :offset="2">
            <div class="calculator-general">
              <h3>Ingresa cantidad</h3>
              <el-row class="calculator-buttons">
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('1')">1</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('2')">2</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('3')">3</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('4')">4</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('5')">5</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('6')">6</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('7')">7</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('8')">8</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('9')">9</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('.')">.</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="addNumber('0')">0</el-button>
                </el-col>
                <el-col class="buttonc" :span="8">
                  <el-button @click="clearPayment()">Correguir</el-button>
                </el-col>
              </el-row>
            </div>
          </el-col>
          <el-col :span="7" :offset="2">
            <div class="finished-sale">
              <p>Número de cliente</p>
              <el-input v-model="form.client_id"></el-input>
              <div class="button">
                <el-button @click="successSale()" type="success">Terminar venta</el-button>
              </div>
            </div>

          </el-col>
        </el-row>
      </div>
    </el-dialog>

  </div>


</template>

<script>
  import firebase from 'firebase/app'
  import 'firebase/firestore'

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

    firebase.firestore().enablePersistence()
      .catch(function(err) {
        console.log("Error al activar la persistencia de datos");
      });

  }else{
    instance_firebase = firebase.app()
  }

  let db = instance_firebase.firestore();

  const { Timestamp, GeoPoint } = firebase.firestore;
  export { Timestamp, GeoPoint }


 export default {
    name: 'ok',
    data: () => ({
      search: "",
      sale_products: [],
      filter_products_show: false,
      productsInSale: [],
      products_find_name: [],
      products_find_name_all: [],
      products_find_sku: [],
      products_find_sku_all: [],
      products: [],
      dialogVisible: false,
      dialogVisibleExit: false,
      client_payment: "",
      inputIsVisible: false,
      findActive: false,
      form: {
        client_id: ""
      }
    }),

    mounted() {
      this.focusSearch()
    },

    computed: {

      clientPaymentToInt(){

       if (Number.isNaN(parseFloat(this.client_payment))){
         return 0
       }
       return parseFloat(this.client_payment);
      },

      combineFindAll(){
        let result =  this.products_find_sku_all.concat(this.products_find_name_all);
        if (result.length >= 1){
          this.dialogVisible = true;
          return result;
        }else{
          this.products_find_name_all = [];
          this.products_find_sku_all = [];
          return [];
        }
      },

      sumSale(){
        let sale = this.productsInSale.map(function (product) {
          return product.units * product.price
        });

        return sale.reduce(function(total, num){ return total + num }, 0);
      }
    },

    methods: {

      clearSearch(){
        this.search = "";
        this.focusSearch();
      },

      handleClose(done) {
        this.focusSearch();
        done();
      },

      handleChange(){
        this.focusSearch();
      },

      focusSearch(){
        this.$refs.search.focus();
      },

      clearPayment(){

        this.client_payment = 0;
      },

      addNumber(number){
        this.client_payment = this.client_payment + number;
      },

      addLetter(letter){
        this.search = this.search + letter;
      },

      initAllVariable(){
          this.search = "";
          this.sale_products = [];
          this.filter_products_show = false;
          this.productsInSale = [];
          this.products_find_name = [];
          this.products_find_name_all = [];
          this.products_find_sku =  [];
          this.products_find_sku_all = [];
          this.products =  [];
          this.dialogVisible =  false;
          this.dialogVisibleExit = false;
          this.client_payment = 0;
          this.focusSearch();
      },

      testQuery(){

        let data = {};
        let products = [];

        //First you get users data
        let document = db.collection("sales");

        document.get().then((snapshot) => {

            //In this case I will store data in some object, so I can add events as an array for a key in each user object

            snapshot.forEach((sale) => {
              let current_sale = sale.data();
              console.log('current_sale', sale.data());
              for (let i=0; i < current_sale.products.length; i++ ){
                console.log('current_sale.products['+i+']._product ', current_sale.products[i].sku);
                if (!products.find(element => element === current_sale.products[i].sku)){
                  products.push({
                    'sku': current_sale.products[i].sku,
                    'count': current_sale.products[i].units
                  });
                }
              }

              /** Valid data persist **/
              let source = snapshot.metadata.fromCache ? "local cache" : "server";
              console.log("Data came from " + source);


            });

          // Seguir aqui

        });

        // console.log(data);

      },

      async successSale() {
        let object_products = this.productsInSale;
        console.log('object_products', object_products);
        let products = [];
        for (let i = 0; i < this.productsInSale.length; i++) {
          let single_product = {
            '_product': db.doc('products/' + this.productsInSale[i].id),
            'sku': this.productsInSale[i].sku,
            'units': this.productsInSale[i].units
          };
          products.push(single_product);
        }

        let sale = {
          'products': products,
          'created_at': firebase.firestore.Timestamp.fromDate(new Date(Date.now())),
          'total': this.sumSale
        };

        db.collection('sales').add(sale);

        this.$notify({
          title: 'Venta finalizada',
          message: 'La venta ha sido guardada',
          type: 'success'
        });

        this.initAllVariable();

      },

      finishSale(){

        this.dialogVisibleExit = true;

      },

      async findProductBySKU() {
        /**
         *  The variable is initialize with empty value.
         * */
        this.products_find_sku = [];

        if (this.search.length >= 4) {
          this.findActive = true;
          await db.collection('products')
            .where('sku', "==", this.search.toUpperCase())
            .get()
            .then((querySnapshot) => {
              querySnapshot.forEach((doc) => {
                let element = doc.data();
                element.id = doc.id;
                this.products_find_sku.unshift(element);
              });

              if(this.products_find_sku.length === 0){
                db.collection('products-not-register').add({
                  'sku': this.search.toUpperCase()
                });
                this.search = "";
                this.findActive = false;
                this.$notify({
                  title: 'No existe producto',
                  message: 'El producto no se encuentra registrado',
                  type: 'error'
                });
              }

              /** Add new element in sale **/

              if (this.products_find_sku.length === 1) {
                this.addProductToSale(
                  this.products_find_sku[0].name,
                  this.products_find_sku[0].price,
                  this.products_find_sku[0].sku,
                  this.products_find_sku[0].id
                );
                this.products_find_name = [];
                this.products_find_sku = [];
                this.search = '';
                this.findActive = false;
                this.focusSearch();
              }
            });

          this.findActive = false;
        }
      },

      onSubmit(event){
        event.stopPropagation();
        event.preventDefault();
        if (this.search.length >= 2){
          this.findProductBySKU();
        }else{
          this.$message.error('Agrega al menos 2 carácteres para realizar la busqueda.');
        }
      },


      async findProductsByName() {

        if (this.search.length >= 2) {
          this.findActive = true;
          await db.collection('products')
            .where('sku', "==", this.search)
            .get()
            .then(querySnapshot => {
              this.products_find_sku_all = querySnapshot.docs.map(doc => doc.data())
            });

          await db.collection('products')
            .where('name', ">=", this.search.toUpperCase())
            .limit(30)
            .get()
            .then(querySnapshot => {
              this.products_find_name_all = querySnapshot.docs.map(doc => doc.data())
            });

          this.findActive = false;

        }else{

          this.$message.error('Agrega al menos 2 carácteres para realizar la busqueda.');

        }

      },


      addProductToSale(name, price, sku,key){
        let new_product = {
          'id': Date.now(),
          'name': name,
          'price': price,
          'units': 1,
          'subtotal': price,
          'sku': sku,
          'key': key
        };
        this.productsInSale.unshift(new_product);
        this.focusSearch();
      },

      addProductButton(index, rows) {
        this.addProductToSale(
          rows[index].name,
          rows[index].price,
          rows[index].sku,
          rows[index].id
        );
        this.dialogVisible = false;
        this.search = "";
      },

      DeleteOfSale: function(id){
        for (let i=0; i < this.productsInSale.length; i++){
          if (this.productsInSale[i].id === id){
            this.productsInSale.splice( i, 1 );
          }
        }
        this.focusSearch();
      },

    },

  }
</script>

<style>

  .my_keyboard_two .el-button{
    margin: 5px;
  }

  .line_keyboard_b_two{
    margin-left: 10px;
  }

  .line_keyboard_c_two{
    margin-left: 15px;
  }

  .search_input input{
    text-transform: uppercase;
  }

  .calculator-buttons{
    text-align: center;
  }

  .calculator-buttons .buttonc{
    padding: 15px 5px;
  }

  .calculator-buttons .buttonc button{
    width: 100%;
  }

  .calculator-general{
    padding: 15px;
  }

  .products_detail {
    font-weight: bold;
    text-align: center;
    padding: 15px;
  }
  .products_table{
    padding: 15px 0px;
    text-align: center;
  }

  .products_table:nth-child(2){
    background: rgba(0, 186, 255, 0.61);
    color: white;
  }

  .total_sale{
    background: #4ca1ff;
    color: white;
    padding: 10px;
    font-size: 20px;
  }
  .search_box{
    color: grey;
    opacity: .7;
  }

  .table_sale{
    padding: 10px;
    background: #f4f4f4;
  }

  .total_container{
    text-align: center;
  }

  .total_box{
    padding: 10px 5px;
    background: #ff5c0f;
    color: white;
    font-size: 30px;
    text-align: center;
  }

  .total_box .el-input__inner{
    padding: 5px 5px;
    background: #ff7300;
    color: white;
    font-size: 30px;
    text-align: center;
  }

  .finished-sale{
    padding: 15px;
  }

  .finished-sale .button{
    padding: 25px 0px;
  }

  .finished-sale .button button{
    width: 100%;
  }

  .space-box{
    padding: 15px 15px;
  }

  .my_keyboard_two span{
    text-transform: uppercase;
  }
</style>
