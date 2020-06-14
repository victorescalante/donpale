<template>
  <div>
    <div id="bg">
      <div class="text-bg">
        <h1>Bienvenido a Tienda Don Pale</h1>
      </div>
      <el-row :gutter="24" class="aling-buttons">
        <el-col :span="8">
          <h2>Punto de venta</h2>
          <nuxt-link to="/sale/point">
            <el-button type="primary">Punto de Venta</el-button>
          </nuxt-link>
        </el-col>
        <el-col :span="8">
          <h2>Productos</h2>
          <nuxt-link to="/products">
            <el-button type="primary">Administrar productos</el-button>
          </nuxt-link>
        </el-col>
        <el-col :span="8">
          <h2>Proveedores</h2>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<style>

  #bg .text-bg{
    text-align: center;
    font-size: 50px;
    color: #ff7300;
    text-shadow: 3px 3px #e9e9e9;
  }

  .aling-buttons{
    text-align: center;
  }

</style>
