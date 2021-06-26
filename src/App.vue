<template>
  <div id="app">
    <div class="barra-nav">
      <!-- Barra de navegacion --------------------------------------------------------------------------------------->
      <b-navbar variant="faded" type="light">
        <b-navbar-brand class="mx-auto" href="#">
          <img src="./assets/Logo_Mesa-de-trabajo-1.jpg" width="125" height="80" alt="Logo Caja Morelia">
        </b-navbar-brand>
      </b-navbar>
    </div>
    <b-container class="mt-5 footer-min">
          <b-table :items="items" :fields="fields" striped responsive="sm">
            <!-- Mostrar detalles ---------------------------------------------------------------------------------->
              <template #cell(ver_detalles)="row">
                  <b-button  variant="success" size="sm" @click="row.toggleDetails(); obtenerDetalles(row.item.idCliente)" class="mr-2">
                    <div style="font-size: 20px;">
                        <b-icon icon="eye" class=" bg-success" variant="light"></b-icon>
                      </div>
                  </b-button>
              </template>
              
              <template #row-details="row">
                <b-card class="text-center">
                  <b-row class="mb-2">
                    <b-col sm="3" class="text-sm-right"><b>Tipo de cuenta:</b></b-col>
                    <b-col>{{ obtenerNombreCuenta(row.item.idCliente)}}</b-col>
                  </b-row>

                  <b-row class="mb-2">
                    <b-col sm="3" class="text-sm-right"><b>Saldo Actual:</b></b-col>
                    <b-col>${{ obtenerSaldoActual(row.item.idCliente)}}</b-col>
                  </b-row>

                  <b-row class="mb-2">
                    <b-col sm="3" class="text-sm-right"><b>Fecha de Ultimo Movimiento:</b></b-col>
                    <b-col>{{ obtenerfecha(row.item.idCliente)}}</b-col>
                  </b-row>

                  <b-button size="sm" @click="row.toggleDetails">Ocultar detalles</b-button>
                </b-card>
              </template>

              <!-- Editar detalles --------------------------------------------------------------------------------->
              <template #cell(editar)="row">
                <b-button v-b-modal.modal-prevent-closing @click="encontrarCliente(row.item.idCliente)" variant="warning" size="sm" class="mr-2">
                    <div style="font-size: 20px;">
                      <b-icon icon="pencil" class=" bg-warning" variant="light"></b-icon>
                    </div>
                </b-button>
              </template>

              <!-- Eliminar registro ------------------------------------------------------------------------------->
              <template #cell(eliminar)="row" class="row justify-content-center">
                <b-button v-b-modal.modal-delete @click='idClient = row.item.idCliente' variant="danger" size="sm" class="mr-2">
                    <div style="font-size: 20px;">
                      <b-icon icon="x-circle" class=" bg-danger" variant="light"></b-icon>
                    </div>
                </b-button>
              </template>

          </b-table>
          <!-- Modal delete ---------------------------------------------------------------------------------------->
          <b-modal  ok-variant = 'danger' ok-title = 'Eliminar' cancel-title ='Cancelar' 
            id="modal-delete"
            ref="modal"
            title="Eliminar"
            @ok="handleOkdelete"
          >
          <p>¿Desea eliminar permanentemente su cliente y sus registros?</p>
          </b-modal>
          <!-- Modal update ---------------------------------------------------------------------------------------->
          <b-modal ok-variant = 'warning' ok-title = 'Aceptar cambios' cancel-title ='Cancelar' 
                id="modal-prevent-closing"
                ref="modal"
                title="Actualiza tus datos"
                @show="resetModal"
                @hidden="resetModal"
                @ok="handleOk"
                >
                <form ref="form" @submit.stop.prevent="handleSubmit">
                  <!-- Input name ---------------------------------------------------------------------------------->
                  <b-form-group
                    label="Nombre"
                    label-for="name-input"
                    invalid-feedback="El nombre es requerido"
                    :state="infoClient.nombreStatus"
                  >
                    <b-form-input
                      id="name-input"
                      v-model="infoClient.nombre"
                      :state="infoClient.nombreStatus"
                      required
                    ></b-form-input>
                  </b-form-group>
                  <!-- Input apeliidopaterno ---------------------------------------------------------------------->
                  <b-form-group
                    label="Apellido Paterno"
                    label-for="apP-input"
                    invalid-feedback="El apellido Paterno es requerido"
                    :state="infoClient.apellidoMaternoStatus"
                  >
                    <b-form-input
                      id="apP-input"
                      v-model="infoClient.apellidoPaterno"
                      :state="infoClient.apellidoPaternoStatus"
                      required
                    ></b-form-input>
                  </b-form-group>
                  <!-- input apellido materno ---------------------------------------------------------------------->
                  <b-form-group
                    label="Apellido Materno"
                    label-for="apM-input"
                    invalid-feedback="El Apellido Materno es requerido"
                    :state="infoClient.apellidoMaternoStatus"
                  >
                    <b-form-input
                      id="apM-input"
                      v-model="infoClient.apellidoMaterno"
                      :state="infoClient.apellidoMaternoStatus"
                      required
                    ></b-form-input>
                  </b-form-group>
                  <!-- input rfc ----------------------------------------------------------------------------------->
                  <b-form-group
                    label="RFC"
                    label-for="rfc-input"
                    invalid-feedback="El rfc es requerido"
                    :state="infoClient.rfcStatus"
                  >
                    <b-form-input
                      id="rfc-input"
                      v-model="infoClient.rfc"
                      :state="infoClient.rfcStatus"
                      required
                    ></b-form-input>
                  </b-form-group>
                  <!-- input curp --------------------------------------------------------------------------------->
                  <b-form-group
                    label="CURP"
                    label-for="curp-input"
                    invalid-feedback="La CURP es requerida"
                    :state="infoClient.curpStatus"
                  >
                    <b-form-input
                      id="curp-input"
                      v-model="infoClient.curp"
                      :state="infoClient.curpStatus"
                      required
                    ></b-form-input>
                  </b-form-group>
                </form>
            </b-modal>
    </b-container>
    <footer class="footer-cv">
    </footer>
  </div>
</template>

<script>
const moment = require('moment')

export default {

  data() {
        return {
          // Declarando variables usadas en los métodos
          idClient:null,
          fields: ['idCliente','nombre', 'apellidoPaterno','apellidoMaterno', 'ver_detalles','editar','eliminar'],
          items: [],
          details:[],
          infoClient:[{
            idCliente:null,
            nombre:'', 
            nombreStatus:null,
            apellidoPaterno:'',
            apellidoPaternoStatus:null,
            apellidoMaterno:'',
            apellidoMaternoStatus:null,
            rfc:'',
            rfcStatus:null,
            curp:'',
            curpStatus:null,

          }]
        }
      },
  beforeMount(){ //Mostramos por default los elementos que arroja el método obtenerClientes()
        this.obtenerClientes();
  },
  methods: {
      formatoFecha(date){  //Usando moment le damos un formato a la fecha que obtenemos de la bd
          moment.locale('es_mx')
          return moment(date).format('lll')
      },
      async obtenerClientes(){ //Accedemos a la ruta principal de la API para obtener todos los registros
          try{
            const response = await fetch('http://localhost:9000/clientes', {
                method : 'GET',
              })
              this.items = await response.json() 
            }catch(err){
              console.log(err)
          }
      },
      async obtenerDetalles(idCliente){ //Obtenemos los datos financieros de cada cliente para guardarlos en un array dinamico y posteriormente consultarlos
          try{
              const response = await fetch(`http://localhost:9000/clientes/verDetalles/${idCliente}`, {
                method : 'GET',
              })
              var elementos = await response.json()
              elementos['idCliente'] = idCliente
              this.details.push(elementos) 
              }catch(err){
              console.log(err)
          }
      },
      obtenerNombreCuenta(idCliente){ //Obtenemos el campo nombreCuenta de nuestro array mencionado anteriormente
        for(var i in this.details){
          const informacion = this.details[i]
          if(informacion.idCliente == idCliente){
              return informacion.nombreCuenta
          }
        }
        return 
      },
      obtenerSaldoActual(idCliente){ //Obtenemos el saldoActual
        for(var i in this.details){
          const informacion = this.details[i]
          if(informacion.idCliente == idCliente){
              return informacion.saldoActual
          }
        }
        return 
      },
      obtenerfecha(idCliente){ //Obtenemos el campo fechaUltimoMovimiento
        for(var i in this.details){
          const informacion = this.details[i]
          if(informacion.idCliente == idCliente){
              return this.formatoFecha(informacion.fechaUltimoMovimiento)
          }
        }
        return 
      },
      async eliminarCliente(idCliente){ //Accedemos a la API para eliminar el elemento seleccionado
          try{
              const response = await fetch(`http://localhost:9000/clientes/eliminar/${idCliente}`, {
                method : 'DELETE',
              })
              console.log(response)
              this.obtenerClientes()
              }catch(err){
              console.log(err)
          }
      },
      async encontrarCliente(idCliente){ //Accedemos a la API para obtener los datos del cliente, mostrarlos en el modal de "editar"
          try{
              const response = await fetch(`http://localhost:9000/clientes/verCliente/${idCliente}`, {
                method : 'GET',
              })
              const client = await response.json()
              this.infoClient = {
                ...this.infoClient, 
                  idCliente: idCliente,
                  nombre: client.nombre,
                  apellidoPaterno:client.apellidoPaterno,
                  apellidoMaterno: client.apellidoMaterno,
                  rfc: client.rfc,
                  curp: client.curp 
              }
              console.log(this.infoClient)
              }catch(err){
              console.log(err)
          }
      },
      async actualizarCliente(idCliente, nombre, apellidoPaterno,apellidoMaterno, rfc, curp){ //Recibimos los elementos que van a reemplazar los antiguos valores y al consumir la API los actualizamos
          try{
              const body = {
                  nombre:nombre,
                  apellidoPaterno:apellidoPaterno,
                  apellidoMaterno:apellidoMaterno,
                  rfc: rfc,
                  curp: curp
              }
              const response = await fetch(`http://localhost:9000/clientes/editarCliente/${idCliente}`, {
                method : 'PATCH',
                headers: {
                  'Content-Type': 'application/JSON'
                },
                body: JSON.stringify(body)
              })
              console.log(response)
              this.obtenerClientes()
              }catch(err){
              console.log(err)
          }
      },   //Metodos del modal    
      checkFormValidity() { //checamos campos vacios en el fomulario "editar"
        const valid = this.$refs.form.checkValidity()
        this.infoClient.nombreStatus = this.infoClient.nombre != 0 ? true : false
        this.infoClient.apellidoPaternoStatus = this.infoClient.apellidoPaterno != 0 ? true : false
        this.infoClient.apellidoMaternoStatus = this.infoClient.apellidoMaterno != 0 ? true : false
        this.infoClient.rfcStatus = this.infoClient.rfc != 0 ? true : false
        this.infoClient.curpStatus = this.infoClient.curp != 0 ? true : false
        return valid
      },
      resetModal() { //Hacemos un reset a los campos del fomulario
        this.infoClient = {
            idCliente: null,
            nombre:'', 
            nombreStatus:null,
            apellidoPaterno:'',
            apellidoPaternoStatus:null,
            apellidoMaterno:'',
            apellidoMaternoStatus:null,
            rfc:'',
            rfcStatus:null,
            curp:'',
            curpStatus:null,
        }
      },
      handleOk(bvModalEvt) {
        // Prevent modal from closing
        bvModalEvt.preventDefault()
        // Trigger submit handler
        this.handleSubmit()
      },
      handleSubmit() {
        // Exit when the form isn't valid
        if (!this.checkFormValidity()) {
          return
        }
        // hacemos el update de los nuevos valores del formulario
        this.actualizarCliente(this.infoClient.idCliente,this.infoClient.nombre,this.infoClient.apellidoPaterno,this.infoClient.apellidoMaterno,this.infoClient.rfc,this.infoClient.curp)
        // Hide the modal manually
        this.$nextTick(() => {
          this.$bvModal.hide('modal-prevent-closing')
        })
      },
      handleOkdelete(bvModalEvt) { //Indicamos la accion a realizar dependiendo el boton seleccionado en el modal "eliminar"
        // Prevent modal from closing
        bvModalEvt.preventDefault()
        this.eliminarCliente(this.idClient)
        this.$nextTick(() => {
          this.$bvModal.hide('modal-delete')
        })
      }
  }

}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
td,th div{
  text-align: center;
}
.footer-cv {
  height: 150px;
  background: linear-gradient(rgba(123, 177, 32 , 0.5), rgba(0, 0, 0, 0.5)), url("./assets/unnamed.png") center no-repeat;
  background-size: cover;
  border-top: 1px solid rgba(50, 50, 50, 0.3);
  width: 100%;
  bottom: 0px;
}

.tabla{
  background-color: #c8f080;
}
.barra-nav{
  background-color: #7bb120;
}
.footer-min{
  min-height: calc(100vh - 150px - 106px);
}

</style>
