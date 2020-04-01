<template>
  <div class="bg-center" v-loading.fullscreen.lock="activeLoading">
    <nuxt-link to="/">
      <el-button type="primary" icon="el-icon-s-home"></el-button>
    </nuxt-link>
    <div>
      <div>
        <h1>Busqueda de productos</h1>
        <el-row>
          <el-col :span="16">
            <form v-on:submit="onSubmit">
              <el-input placeholder="Digita y/o busca el codigo de barras" ref="search" v-model="search"></el-input>
              <el-input class="button-success-search" type="submit" value="Agregar o actualizar producto"></el-input>
            </form>
          </el-col>
          <el-col :span="8">
            <el-button style="margin-left: 15px" type="secondary" @click.stop="findProductsByName()" icon="el-icon-search">Buscar</el-button>
          </el-col>
        </el-row>
      </div>

      <el-row :gutter="24">
        <el-row class="space-box">
          <el-col :span="24">
            <div class="my_keyboard">
              <div class="line_keyboard_0">
                <el-button @click="addLetter('1')">1</el-button>
                <el-button @click="addLetter('2')">2</el-button>
                <el-button @click="addLetter('3')">3</el-button>
                <el-button @click="addLetter('4')">4</el-button>
                <el-button @click="addLetter('5')">5</el-button>
                <el-button @click="addLetter('6')">6</el-button>
                <el-button @click="addLetter('7')">7</el-button>
                <el-button @click="addLetter('8')">8</el-button>
                <el-button @click="addLetter('9')">9</el-button>
                <el-button @click="addLetter('0')">0</el-button>
              </div>
              <div class="line_keyboard_a">
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
              <div class="line_keyboard_b">
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
              <div class="line_keyboard_c">
                <el-button @click="addLetter('z')">z</el-button>
                <el-button @click="addLetter('x')">x</el-button>
                <el-button @click="addLetter('c')">c</el-button>
                <el-button @click="addLetter('v')">v</el-button>
                <el-button @click="addLetter('b')">b</el-button>
                <el-button @click="addLetter('n')">n</el-button>
                <el-button @click="addLetter('m')">m</el-button>
                <el-button @click="addLetter(' ')" style="padding: 15px 50px">Espacio</el-button>
                <el-button :disabled="search <= 0" type="danger" @click="clearSearch()">Limpiar</el-button>
              </div>
            </div>
          </el-col>
        </el-row>
      </el-row>
    </div>

    <el-dialog
      title="Busqueda de productos"
      :visible.sync="dialogVisible"
      fullscreen>
      <div>
        <el-table
          stripe
          :data="products_find_name_all"
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
              <el-button @click="handleEdit(scope.$index, scope.row)">
                Editar Producto
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-dialog>

    <el-dialog
      :title="actionTitle()"
      :visible.sync="editVisible"
      fullscreen
    >
      <el-form :model="form" :rules="rules" ref="ruleForm">
        <el-row class="form_style">
          <el-col :span="10">
            <el-form-item label="SKU del producto" prop="sku">
              <el-input :disabled="disableSku(form.sku)" v-model="form.sku"></el-input>
            </el-form-item>
            <el-form-item label="Nombre del producto" prop="name">
              <el-input v-model="form.name"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="Costo (En cuanto fue comprado)" prop="cost">
              <el-input-number v-model="form.cost" :precision="1" :step="0.5" :min="0"></el-input-number>
            </el-form-item>
            <el-form-item label="Precio (En cuanto será vendido)" prop="price">
              <el-input-number v-model="form.price" :precision="1" :step="0.5" :min="0"></el-input-number>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <div class="padding-top-15">
              <el-button type="primary" @click="submitForm('ruleForm')" v-if="product_id.length > 0">Actualizar Producto</el-button>
              <el-button type="primary" @click="submitForm('ruleForm')" v-if="product_id.length <= 0">Crear Producto</el-button>
            </div>
          </el-col>
        </el-row>
      </el-form>

      <el-row :gutter="24">
        <el-row class="space-box">
          <el-col :span="24">
            <div class="my_keyboard small_k">
              <div class="line_keyboard_0">
                <el-button @click="addLetterForm('1')">1</el-button>
                <el-button @click="addLetterForm('2')">2</el-button>
                <el-button @click="addLetterForm('3')">3</el-button>
                <el-button @click="addLetterForm('4')">4</el-button>
                <el-button @click="addLetterForm('5')">5</el-button>
                <el-button @click="addLetterForm('6')">6</el-button>
                <el-button @click="addLetterForm('7')">7</el-button>
                <el-button @click="addLetterForm('8')">8</el-button>
                <el-button @click="addLetterForm('9')">9</el-button>
                <el-button @click="addLetterForm('0')">0</el-button>
                <el-button :disabled="form.name <= 0" type="danger" @click="removeLetterForm()"> < Coreguir</el-button>
              </div>
              <div class="line_keyboard_a">
                <el-button @click="addLetterForm('q')">q</el-button>
                <el-button @click="addLetterForm('w')">w</el-button>
                <el-button @click="addLetterForm('e')">e</el-button>
                <el-button @click="addLetterForm('r')">r</el-button>
                <el-button @click="addLetterForm('t')">t</el-button>
                <el-button @click="addLetterForm('y')">y</el-button>
                <el-button @click="addLetterForm('u')">u</el-button>
                <el-button @click="addLetterForm('i')">i</el-button>
                <el-button @click="addLetterForm('o')">o</el-button>
                <el-button @click="addLetterForm('p')">p</el-button>
              </div>
              <div class="line_keyboard_b">
                <el-button @click="addLetterForm('a')">a</el-button>
                <el-button @click="addLetterForm('s')">s</el-button>
                <el-button @click="addLetterForm('d')">d</el-button>
                <el-button @click="addLetterForm('f')">f</el-button>
                <el-button @click="addLetterForm('g')">g</el-button>
                <el-button @click="addLetterForm('h')">h</el-button>
                <el-button @click="addLetterForm('j')">j</el-button>
                <el-button @click="addLetterForm('k')">k</el-button>
                <el-button @click="addLetterForm('l')">l</el-button>
                <el-button @click="addLetterForm('ñ')">ñ</el-button>
              </div>
              <div class="line_keyboard_c">
                <el-button @click="addLetterForm('z')">z</el-button>
                <el-button @click="addLetterForm('x')">x</el-button>
                <el-button @click="addLetterForm('c')">c</el-button>
                <el-button @click="addLetterForm('v')">v</el-button>
                <el-button @click="addLetterForm('b')">b</el-button>
                <el-button @click="addLetterForm('n')">n</el-button>
                <el-button @click="addLetterForm('m')">m</el-button>
                <el-button @click="addLetterForm('.')">.</el-button>
                <el-button @click="addLetterForm(' ')" style="padding: 15px 50px">Espacio</el-button>
              </div>
            </div>
          </el-col>
        </el-row>
      </el-row>

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

  }else{
    instance_firebase = firebase.app()
  }

  let db = instance_firebase.firestore();

  const { Timestamp, GeoPoint } = firebase.firestore;
  export { Timestamp, GeoPoint }

  export default {
    data() {
      return {
        search: '',
        products_find_sku: [],
        products_find_name_all: [],
        dialogVisible: false,
        editVisible: false,
        activeLoading: false,
        product_id: "",
        form: {
          name: "",
          sku: "",
          cost: "",
          price: "",
          is_basic: false,
          seller: "central"
        },
        rules: {
          name: [
            { required: true, message: 'Agrega un nombre', trigger: 'blur' },
          ],
          cost: [
            { required: true, message: 'Agrega el costo del producto', trigger: 'blur'},
          ],
          price: [
            { required: true, message: 'Agrega el costo del producto', trigger: 'blur'},
          ]
        }
      }
    },

    methods: {

      clearNameProduct(){
        this.form.name = "";
      },

      actionTitle(){

        if (this.product_id.length === 0){
          return "Crear Producto";
        }
        return "Actualizar Producto";

      },

      addLetter(letter){
        this.search = this.search + letter;
      },

      addLetterForm(letter){
        this.form.name = this.form.name + letter;
      },

      removeLetterForm(){
        this.form.name = this.form.name.substring(0, this.form.name.length - 1);
      },

      clearSearch(){
        this.search = "";
        this.focusSearch();
      },

      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (!valid) {
            this.$message('Datos de formulario incorrecto');
            return false;
          } else {

            this.form.name = this.form.name.toUpperCase();
            this.activeLoading = true;

            if (this.product_id.length === 0){

              db.collection('products').add(this.form).then(function () {
                this.$message.success('Guardado correctamente');
                this.form = {
                  name: "",
                    sku: "",
                    cost: "",
                    price: "",
                    is_basic: false,
                    seller: "central"
                };
                this.editVisible = false;
                this.search = "";
                this.activeLoading = false;
              }.bind(this)).catch(function (error) {
                this.$message.error('Error al guardar producto');
                this.activeLoading = false;
              }.bind(this))

            } else{
              db.collection('products').doc(this.product_id).set(this.form).then(function () {
                this.$message.success('Actualizado correctamente');
                this.form = {
                  name: "",
                  sku: "",
                  cost: "",
                  price: "",
                  is_basic: false,
                  seller: "central"
                };
                this.editVisible = false;
                this.search = "";
                this.activeLoading = false;
              }.bind(this)).catch(function (error) {
                this.$message.error('Error al actualizar');
                this.activeLoading = false;
              }.bind(this));
            }
          }
        });
      },
      disableSku(sku){
        return sku.length !== 0;
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

      focusSearch(){
        this.$refs.search.focus();
      },

      handleEdit(index, row) {
        this.dialogVisible = false;
        this.editProduct(row)
      },

      async findProductBySKU() {
        /**
         *  The variable is initialize with empty value.
         * */
        this.products_find_sku = [];

        if (this.search.length >= 2) {
          this.activeLoading = true;
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
                this.addProduct(this.search.toUpperCase());
                this.activeLoading = false;
              }

              /** Add new element in sale **/

              if (this.products_find_sku.length === 1) {
                console.log('product_firestore', this.products_find_sku[0]);
                this.editProduct(this.products_find_sku[0]);
                this.activeLoading = false;
              }

              this.activeLoading = false;
            });
        }
      },

      editProduct(product){
        this.editVisible = true;

        // Add element product to form
        this.product_id = product.id;
        this.form.cost = product.cost;
        this.form.sku = product.sku;
        this.form.name = product.name;
        this.form.price = product.price;
        this.form.seller = 'central';
        this.form.is_basic = false;

      },

      addProduct(sku){
        this.editVisible = true;

        // Add element product to form
        this.product_id = "";
        this.form.cost = 0;
        this.form.sku = sku;
        this.form.name = "";
        this.form.price = 0;
        this.form.seller = 'central';
        this.form.is_basic = false;

      },


      async findProductsByName() {

        if (this.search.length >= 2) {

          this.activeLoading = true;
          await db.collection('products')
            .where('name', ">=", this.search.toUpperCase())
            .limit(30)
            .get()
            .then(querySnapshot => {
              this.products_find_name_all = querySnapshot.docs.map(doc => {
                let element = doc.data();
                element.id = doc.id;
                return element;
                }
              )
            });

          this.dialogVisible = true;
          this.activeLoading = false;
        }else{
          this.activeLoading = false;
          this.$message.error('Agrega al menos 2 carácteres para realizar la busqueda.');

        }

      },

    }
  }
</script>

<style>

  .button-success-search {
    padding: 25px 0px
  }

  .button-success-search input{
    background: #1568ff;
    color: white;
  }

  .bg-center span{
    text-transform: uppercase;
  }

  .bg-center input{
    text-transform: uppercase;
  }

  .my_keyboard{
    text-align: center;
    background: rgba(211, 211, 211, 0.27);
    border-radius: 5px;
  }

  .line_keyboard_0{
    margin-left: 0px;
  }

  .my_keyboard .el-button{
    margin: 10px;
  }

  .my_keyboard.small_k .el-button{
    margin: 5px;
  }

  .line_keyboard_a{
    margin-left: 10px;
  }

  .line_keyboard_b{
    margin-left: 15px;
  }

  .line_keyboard_c{
    margin-left: 20px;
  }


  .el-button{
    margin-bottom: 10px;
  }

  .padding-top-15{
    padding-top: 35px;
  }
  .form_style > div{
    padding: 15px;
  }
  .bg-center{
    width: 100%;
  }

  .center-search{
    text-align: center;
  }
</style>
