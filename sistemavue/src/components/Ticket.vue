<template>
    <v-layout align-start>
        <v-flex>
            <v-data-table :headers="headers" :items="tickets" :search="search" class="elevation-1">
                <template v-slot:top>
                    <v-toolbar flat>
                        <v-toolbar-title>Tickets</v-toolbar-title>
                        <v-divider class="mx-4" inset vertical></v-divider>
                        <v-spacer></v-spacer>
                        <v-text-field class="text-xs-center" v-model="search" append-icon="search" label="Búsqueda"
                            single-line hide-details></v-text-field>
                        <v-spacer></v-spacer>
                        <v-dialog v-model="dialog" max-width="500px">
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
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
                                            <v-flex xs12 sm6 md6>
                                                <v-text-field v-model="titulo" label="Título"></v-text-field>
                                            </v-flex>
                                            <v-flex xs12 sm6 md6>
                                                <v-text-field v-model="descripcion" label="Descripción"></v-text-field>
                                            </v-flex>
                                            <v-flex xs12 sm6 md6>
                                                <v-select :items="categorias" v-model="tipo_ticket" label="Categoría">
                                                </v-select>
                                            </v-flex>
                                            <v-flex xs12 sm6 md6>
                                                <v-select :items="usuarios" v-model="asignado" label="Asignado">
                                                </v-select>
                                            </v-flex>
                                            <v-flex xs12 sm6 md6>
                                                <v-select :items="personas" v-model="solicitante" label="Solicitante">
                                                </v-select>
                                            </v-flex>
                                            <v-flex xs12 sm6 md6>
                                                <v-select :items="prioridades" v-model="prioridad" label="Prioridad">
                                                </v-select>
                                            </v-flex>
                                            <v-flex xs12 sm6 md6 v-show="valida">
                                                <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">

                                                </div>
                                            </v-flex>
                                        </v-layout>
                                    </v-container>
                                </v-card-text>

                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn color="blue darken-1" text @click="close">
                                        Cancelar
                                    </v-btn>
                                    <v-btn color="blue darken-1" text @click="guardar">
                                        Guardar
                                    </v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>

                        <v-dialog v-model="adModal" max-width="290">
                            <v-card>
                                <v-card-title class="headline" v-if="adAccion == 1">
                                    Activar Ticket
                                </v-card-title>
                                <v-card-title class="headline" v-if="adAccion == 2">
                                    Desactivar Ticket
                                </v-card-title>
                                <v-card-title class="headline" v-if="adAccion == 3">
                                    Cerrar Ticket
                                </v-card-title>
                                <v-card-text>
                                    Estás a punto de <span v-if="adAccion == 1">activar </span>
                                    <span v-if="adAccion == 2">desactivar </span>
                                    <span v-if="adAccion == 3">cerrar </span>
                                    
                                    el ticket {{ adNombre }}
                                </v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn @click="activarDesactivarCerrar()" color="green darken-1" text="text">
                                        Cancelar
                                    </v-btn>
                                    <v-btn v-if="adAccion == 1" @click="activar()" color="orange darken-4" text="text">
                                        Activar
                                    </v-btn>
                                    <v-btn v-if="adAccion == 2" @click="desactivar()" color="orange darken-4" text="text">
                                        Desactivar
                                    </v-btn>
                                    <v-btn v-if="adAccion == 3" @click="cerrar()" color="orange darken-4" text="text">
                                        Cerrar
                                    </v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                    </v-toolbar>
                </template>
                <template v-slot:item.opciones="{ item }">
                    <v-icon small class="mr-2" @click="editItem(item)">
                        edit
                    </v-icon>
                    <template v-if="item.estado">
                        <v-icon small @click="activarDesactivarMostrar(2, item)">
                            delete
                        </v-icon>
                    </template>
                    <template v-else="item.estado">
                        <v-icon small @click="activarDesactivarMostrar(1, item)">
                            check
                        </v-icon>
                    </template>
                    <v-icon small class="mr-2"  @click="activarDesactivarMostrar(3, item)">
                        block
                    </v-icon>
                </template>
                <template v-slot:item.estado="{ item }">
                    <div v-if="item.estado===1">
                        <span class="blue--text">Activo</span>
                    </div>
                    <div v-if="item.estado===0">
                        <span class="red--text">Inactivo</span>
                    </div>
                    <div v-if="item.estado===2">
                        <span class="green--text">Cerrado</span>
                    </div>
                 
                </template>
                <template v-slot:item.createAt="{ item }">
                       {{item.createAt.slice(0,10)}}
                </template>

                <template v-slot:no-data>
                    <v-btn color="primary" @click="listar()">
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
        search: '',
        tickets: [],
        dialogDelete: false,

        headers: [
            { text: 'Opciones', value: 'opciones', sortable: false },
            { text: 'Título', value: 'titulo', sortable: true },
            { text: 'Categoría', value: 'tipo_ticket.nombre', sortable: true },
            { text: 'Descripcion', value: 'descripcion', sortable: true },
            { text: 'Asignado', value: 'asignado.nombre', sortable: false },
            { text: 'Solicitante', value: 'solicitante.nombre', sortable: false },
            { text: 'Prioridad', value: 'prioridad', sortable: false },
             { text: 'Fecha', value: `createAt`, sortable: true },
            { text: 'Estado', value: 'estado', sortable: false },

        ],
        editedIndex: -1,
        _id: '',
        titulo: '',
        descripcion: '',
        tipo_ticket: '',
        categorias: [],
        asignado: '',
        usuarios: [],
        solicitante: '',
        personas: [],
        prioridad: [],
        prioridades: [
            {text:'Urgente', value:'Urgente'},
            {text:'Alta', value:'Alta'},
            {text:'Media', value:'Media'},
            {text:'Baja', value:'Baja'},
         ],
         createAt:'',
        valida: 0,
        validaMensaje: [],
        adModal: 0,
        adAccion: 0,
        adNombre: '',
        adId: ''
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1 ? 'Nuevo Registro' : 'Editar Registro'
        },
    },

    watch: {
        dialog(val) {
            val || this.close()
        },

    },

    created() {
        this.listar()
        this.selectCategorias()
        this.selectAsignado()
        this.selectSolicitante()
    },

    methods: {
        async selectCategorias() {
            try {
                let me = this
                let categoriaArr = []
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                const get = await axios.get('categoria/list', configuracion)
                const data = get.data
                categoriaArr = data
                categoriaArr
                .filter((f)=>{
                   return f.estado===1
                })
                .map((c) => {
                    me.categorias.push({
                        text: c.nombre,
                        value: c._id
                    })
                })
            } catch (error) {
                console.log(error)
            }
        },
        async selectAsignado() {
            try {
                let me = this
                let asignadoArr = []
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                const get = await axios.get('usuario/list', configuracion)
                const data = get.data
                console.log(data)
                asignadoArr = data
                asignadoArr
                    .filter((f) => f.rol !== 'Administrador' && f.rol!=='Cliente' && f.estado===1)
                    .map((c) => {
                        me.usuarios.push({
                            text: c.nombre,
                            value: c._id
                        })
                    })
            } catch (error) {
                console.log(error)
            }
        },
        async selectSolicitante() {
            try {
                let me = this
                let solicitanteArr = []
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                const get = await axios.get('persona/list', configuracion)
                const data = get.data
                solicitanteArr = data
                solicitanteArr
                 .filter((f)=>{
                   return f.estado===1
                })
                .map((c) => {
                    me.personas.push({
                        text: c.nombre,
                        value: c._id
                    })
                })
            } catch (error) {
                console.log(error)
            }
        },

        async listar() {
            try {
                let me = this
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                const get = await axios.get('ticket/list', configuracion)
                const data = get.data
                me.tickets = data
            } catch (error) {
                console.log(error)
            }
        },

        limpiar() {
            this._id = '';
            this.titulo = '';
            this.descripcion = '';
            this.prioridad = '',
            this.createAt='',
            this.valida = 0;
            this.validaMensaje = [];
            this.editedIndex = -1
        },

        validar() {
            this.valida = 0;
            this.validaMensaje = [];
            if (!this.titulo || this.titulo.length > 50) {
                this.validaMensaje.push('Debe contener un título no mayor a 50 caracteres.');
            }
            if (this.descripcion.length > 255) {
                this.validaMensaje.push('La descripcion debe contener máximo 255 caracteres.');
            }
            if (!this.tipo_ticket) {
                this.validaMensaje.push('Debe seleccionar una categoria.');
            }
            if (!this.asignado) {
                this.validaMensaje.push('Debe asignar el ticket a un analista');
            }
            if (!this.solicitante) {
                this.validaMensaje.push('Debe seleccionar su nombre');
            }
            if (!this.prioridad) {
                this.validaMensaje.push('Debe seleccionar una prioridad.');
            }
            if (this.validaMensaje.length) {
                this.valida = 1;
            }
            return this.valida;
        },

        async guardar() {
            try {
                let me = this
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                if (this.validar()) {
                    return
                }
                if (this.editedIndex > -1) {
                    //Editar
                    await axios.put('ticket/update', {
                       '_id': this._id,
                        'titulo': this.titulo,
                        'descripcion': this.descripcion,
                        'tipo_ticket': this.tipo_ticket,
                        'asignado': this.asignado,
                        'solicitante': this.solicitante,
                        'prioridad': this.prioridad,
                    }, configuracion)
                    me.limpiar()
                    me.close()
                    me.listar()
                } else {
                    //Guardar
                    await axios.post('ticket/add', {
                       
                        'titulo': this.titulo,
                        'descripcion': this.descripcion,
                        'tipo_ticket': this.tipo_ticket,
                        'asignado': this.asignado,
                        'solicitante': this.solicitante,
                        'prioridad': this.prioridad,
                    }, configuracion)
                    me.limpiar()
                    me.close()
                    me.listar()

                }

            } catch (error) {
                console.log(error)
            }

        },




        editItem(item) {
            this._id = item._id;
            this.titulo = item.titulo;
            this.descripcion = item.descripcion;
            this.tipo_ticket = item.tipo_ticket._id,
            this.asignado = item.asignado._id,
            this.solicitante = item.solicitante._id,
            this.prioridad = item.prioridad,
            this.dialog = true
            this.editedIndex = 1;
        },

        activarDesactivarMostrar(accion, item) {
            this.adModal = 1
            this.adNombre = item.nombre
            this.adId = item._id

            if (accion == 1) {
                this.adAccion = 1
            } else if (accion == 2) {
                this.adAccion = 2
            } else if (accion == 3) {
                this.adAccion = 3
            }
             else {
                this.addModal = 0
            }
        },

        activarDesactivarCerrar() {
            this.adModal = 0
        },

        async activar() {
            try {
                let me = this
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                await axios.put('ticket/activate', {
                    '_id': this.adId
                }, configuracion)
                me.adModal = 0
                me.adAccion = 0
                me.adNombre = ''
                me.adId = ''
                me.limpiar()
                me.close()
                me.listar()

            } catch (error) {
                console.log(error)
            }
        },
        async desactivar() {
            try {
                let me = this
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                await axios.put('ticket/desactivate', {
                    '_id': this.adId
                }, configuracion)
                me.adModal = 0
                me.adAccion = 0
                me.adNombre = ''
                me.adId = ''
                me.limpiar()
                me.close()
                me.listar()

            } catch (error) {
                console.log(error)
            }
        },
        async cerrar() {
            try {
                let me = this
                let header = { 'Token': this.$store.state.token }
                let configuracion = { headers: header }
                await axios.put('ticket/close', {
                    '_id': this.adId
                }, configuracion)
                me.adModal = 0
                me.adAccion = 0
                me.adNombre = ''
                me.adId = ''
                me.limpiar()
                me.close()
                me.listar()

            } catch (error) {
                console.log(error)
            }
        },

    

        close() {
            this.dialog = false
        },

        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },

    },
}
</script>