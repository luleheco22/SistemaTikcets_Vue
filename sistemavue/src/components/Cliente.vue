<template>
 <v-layout align-start>
<v-flex>
  <v-data-table
    :headers="headers"
    :items="personas"
    :search="search"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Clientes</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
         <v-text-field class="text-xs-center" v-model="search" append-icon="search" 
                label="Búsqueda" single-line hide-details></v-text-field>
                <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              Nuevo
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                 <v-layout wrap>
                                <v-flex xs12 sm12 md12>
                                    <v-text-field v-model="nombre" label="Nombre"></v-text-field>
                                </v-flex>
               
                             
                                <v-flex xs12 sm6 md6>
                                    <v-select v-model="tipo_documento" :items="documentos" label="Tipo Documento"></v-select>
                                </v-flex>
                                <v-flex xs12 sm6 md6>
                                    <v-text-field v-model="num_documento" label="Número Documento"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md6>
                                    <v-text-field v-model="direccion" label="Dirección"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md6>
                                    <v-text-field v-model="telefono" label="Teléfono"></v-text-field>
                                </v-flex>
                                <v-flex xs12 sm6 md6>
                                    <v-text-field v-model="email" label="Email"></v-text-field>
                                </v-flex>
                
                                <v-flex xs12 sm12 md12 v-show="valida">
                                    <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">

                                    </div>
                                </v-flex>
                            </v-layout>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="close"
              >
                Cancelar
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="guardar"
              >
                Guardar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
       
         <v-dialog v-model="adModal" max-width="290">
                    <v-card>
                        <v-card-title class="headline" v-if="adAccion==1">
                            Activar Item
                        </v-card-title>
                        <v-card-title class="headline" v-if="adAccion==2">
                            Desactivar Item
                        </v-card-title>
                        <v-card-text>
                            Estás a punto de <span v-if="adAccion==1">activar </span>
                            <span v-if="adAccion==2">desactivar </span> el item {{adNombre}}
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn @click="activarDesactivarCerrar()" color="green darken-1" text="text">
                                Cancelar
                            </v-btn>
                            <v-btn v-if="adAccion==1" @click="activar()" color="orange darken-4" text="text">
                                Activar
                            </v-btn>
                            <v-btn v-if="adAccion==2" @click="desactivar()" color="orange darken-4" text="text">
                                Desactivar
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.opciones="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        edit
      </v-icon>
      <template v-if="item.estado">
      <v-icon
        small
        @click="activarDesactivarMostrar(2,item)"
      >
        block
      </v-icon>
     </template>
      <template v-else="item.estado">
      <v-icon
        small
        @click="activarDesactivarMostrar(1,item)"
      >
        check
      </v-icon>
     </template>
    </template>
     <template v-slot:item.estado="{ item }">
       <div v-if="item.estado">
                        <span class="blue--text">Activo</span>
                    </div>
                    <div v-else>
                        <span class="red--text">Inactivo</span>
                    </div>
     </template>
    
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="listar()"
      >
        Reset
      </v-btn>
    </template>
  </v-data-table>
  </v-flex>
  </v-layout>
</template>
<script>
import axios from 'axios'
  export default {
    data: () => ({
      dialog: false,
      search:'',
      personas:[],
      dialogDelete: false,

      headers: [
            { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Nombre', value: 'nombre', sortable: true },
                    { text: 'Tipo Persona', value: 'tipo_persona', sortable: true },
                    { text: 'Tipo Documento', value: 'tipo_documento', sortable: true },
                    { text: 'Número Documento', value: 'num_documento', sortable: false  },
                    { text: 'Dirección', value: 'direccion', sortable: false  },
                    { text: 'Teléfono', value: 'telefono', sortable: false  },
                    { text: 'Email', value: 'email', sortable: false  },
                    { text: 'Estado', value: 'estado', sortable: false  } 

      ],
                editedIndex: -1,
                _id:'',
                nombre:'',
               tipo_persona:'Cliente',
                tipo_documento:'',
                documentos: ['DNI','RUC','PASAPORTE','CEDULA','RIF'],
                num_documento: '',
                direccion: '',
                telefono: '',
                email: '',
                valida:0,
                validaMensaje:[],
                adModal:0,
                adAccion:0,
                adNombre:'',
                adId:''
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Nuevo Registro' : 'Editar Registro'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    
    },

    created () {
      this.listar()
    },

    methods: {
       async listar(){
            try {
              let me=this
              let header={'Token':this.$store.state.token}
              let configuracion={headers:header}
                const get= await  axios.get('persona/list',configuracion)
                const data=get.data
                me.personas=data
            } catch (error) {
                console.log(error)
            }
        },

        limpiar(){
                this._id='';
                this.nombre='';
                this.tipo_documento=''
                this.num_documento='';
                this.direccion='';
                this.telefono='';
                this.email='';
                this.valida=0;
                this.validaMensaje=[];
                this.editedIndex=-1;
        },

           validar(){
               this.valida=0;
                this.validaMensaje=[];
              
                if(this.nombre.length<1 || this.nombre.length>50){
                    this.validaMensaje.push('El nombre del usuario debe tener entre 1-50 caracteres.');
                }
                if(!this.tipo_persona){
                    this.validaMensaje.push('Debe seleccionar un tipo de persona.');
                }
                if(this.num_documento.length>20){
                    this.validaMensaje.push('El documento no debe tener más de 20 caracteres.');
                }
                if(this.direccion.length>70){
                    this.validaMensaje.push('La dirección no debe tener más de 70 caracteres.');
                }
                if(this.telefono.length>20){
                    this.validaMensaje.push('El teléfono no debe tener más de 20 caracteres.');
                }
                if(this.email.length<1 || this.nombre.length>50){
                    this.validaMensaje.push('El email del usuario debe tener entre 1-50 caracteres.');
                }
              
                if (this.validaMensaje.length){
                    this.valida=1;
                }
                return this.valida;
            },

       async guardar(){
        try {
            let me=this
            let header={'Token':this.$store.state.token}
              let configuracion={headers:header}
            if (this.validar()) {
                return
            }
            if (this.editedIndex>-1) {
                //Editar
                  await  axios.put('persona/update',{
                    '_id':this._id,
                        'tipo_documento':this.tipo_documento,
                        'nombre':this.nombre,
                        'tipo_documento':this.tipo_documento,
                        'num_documento':this.num_documento,
                        'direccion':this.direccion,
                        'telefono': this.telefono,
                        'email':this.email,
                      
              },configuracion)
              me.limpiar()
              me.close()
              me.listar()
            }else{
                //Guardar
              await  axios.post('persona/add',{
                       'tipo_persona':this.tipo_persona,
                        'nombre':this.nombre,
                        'tipo_documento':this.tipo_documento,
                        'num_documento':this.num_documento,
                        'direccion':this.direccion,
                        'telefono': this.telefono,
                        'email':this.email,
                      
              },configuracion)
              me.limpiar()
              me.close()
              me.listar()
             
            }
            
        } catch (error) {
           console.log(error)   
        }

        },


     

      editItem (item) {
        this._id=item._id;
                this.tipo_documento=item.tipo_documento;
                this.nombre=item.nombre;
                this.tipo_documento=item.tipo_documento;
                this.num_documento=item.num_documento;
                this.direccion=item.direccion;
                this.telefono=item.telefono;
                this.email=item.email;
                this.password=item.password;
                this.dialog = true;
                this.editedIndex=1;
      },

     activarDesactivarMostrar(accion,item){
             this.adModal=1
             this.adNombre=item.nombre
             this.adId=item._id

              if (accion==1) {
               this.adAccion=1
              }else if (accion==2) {
                this.adAccion=2
              }else{
                 this.addModal=0
              }
     },

     activarDesactivarCerrar(){
        this.adModal=0
     },

    async activar(){
        try {
            let me=this
            let header={'Token':this.$store.state.token}
              let configuracion={headers:header}
             await  axios.put('persona/activate',{
                    '_id':this.adId
              },configuracion)
              me.adModal=0
              me.adAccion=0
              me.adNombre=''
              me.adId=''
              me.limpiar()
              me.close()
              me.listar()
            
        } catch (error) {
            console.log(error)
        }
     },
    async desactivar(){
        try {
            let me=this
            let header={'Token':this.$store.state.token}
              let configuracion={headers:header}
             await  axios.put('persona/desactivate',{
                    '_id':this.adId
              },configuracion)
              me.adModal=0
              me.adAccion=0
              me.adNombre=''
              me.adId=''
              me.limpiar()
              me.close()
              me.listar()
            
        } catch (error) {
            console.log(error)
        }
     },


      close () {
        this.dialog = false
      },


    },
  }
</script>