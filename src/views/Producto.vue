<template>
  <div>
    <b-row>
      <b-col cols="12">
        <b-card border-variant="primary" header-bg-variant="primary" :header="'<strong>Gestionar Productos</strong> (' + totalFilas + ')'" tag="article" class="m-3 mt-3">
          <b-row class="mt-1 mb-2">
            <b-col cols="12">
              <b-form-group>
                <b-input-group cols="9">
                  <b-form-input
                    type="text"
                    v-model="filtro"
                    placeholder="Filtrar Búsqueda"
                    autocomplete="text"
                  ></b-form-input>
                  <!-- Attach Right button -->
                  <b-input-group-append>
                    <b-button variant="primary" :disabled="!filtro" @click.stop="filtro = ''">x</b-button>
                    <b-btn
                      class="ml-3"
                      @click.stop="cargarFormulario(null,'Agregar')"
                      variant="primary"
                    >Nuevo</b-btn>
                  </b-input-group-append>
                </b-input-group>
              </b-form-group>
            </b-col>
          </b-row>

          <b-row align-h="center">
            <b-col class="contenedor-tabla">
              <b-table
                v-if="items && items.length > 0"
                stacked="md"
                striped
                hover
                class="tabla"
                :fields="campos"
                :items="items"
                :current-page="paginaActual"
                :per-page="porPagina"
                :filter="filtro"
                @filtered="actualizarTabla"
              >
                <template slot="acciones" slot-scope="row">
                  <b-button
                    style="margin: 1px;"
                    size="sm"
                    @click.stop="cargarFormulario(row.item,'Ver')"
                  >
                    <i class="icon-eye"></i>
                  </b-button>
                  <b-button
                    style="margin: 1px;"
                    size="sm"
                    variant="primary"
                    @click.stop="cargarFormulario(row.item,'Modificar')"
                  >
                    <i class="icon-pencil"></i>
                  </b-button>
                  <b-button
                    style="margin: 1px;"
                    size="sm"
                    variant="danger"
                    @click.stop="cargarFormulario(row.item,'Eliminar')"
                  >
                    <i class="fa fa-trash"></i>
                  </b-button>
                </template>
              </b-table>
              <b-alert v-else show>No se encontraron registros.</b-alert>
            </b-col>
          </b-row>

          <b-row class="mb-5">
            <b-col>
              <b-pagination
                align="center"
                :total-rows="totalFilas"
                :per-page="porPagina"
                v-model="paginaActual"
                class="my-0"
              />
            </b-col>
          </b-row>

          <b-modal v-if="objeto" centered v-model="showModal" :title="tipoOperacion">
            <b-container>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Descripción
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.descripcion}}
                  </span>
                  <b-form-input
                    v-else
                    type="text"
                    required
                    v-model="objeto.descripcion"
                    class="form-control"/>
                </b-col>
              </b-row>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Orden
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.orden}}
                  </span>
                  <b-form-input
                    v-else
                    type="number"
                    required
                    v-model="objeto.orden"
                    class="form-control"/>
                </b-col>
              </b-row>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Costo
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.costo}}
                  </span>
                  <money v-else v-model="objeto.costo" class="form-control"></money>
                </b-col>
              </b-row>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Precio
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.precio}}
                  </span>
                  <money v-else v-model="objeto.precio" class="form-control"></money>
                </b-col>
              </b-row>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Cantidad
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.cantidad}}
                  </span>
                  <b-form-input
                    v-else
                    type="number"
                    required
                    v-model="objeto.cantidad"
                    class="form-control"/>
                </b-col>
              </b-row>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Estado
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.estado}}
                  </span>
                  <b-form-select v-else v-model="objeto.estado" class="mb-3">
                    <option :key="'esta_' + e" v-for="(estado, e) in estados" :value="estado.descripcion">{{estado.descripcion}}</option>
                  </b-form-select>
                </b-col>
              </b-row>
              <b-row class="mb-3">
                <b-col
                  class="text-left">
                  <span class="font-weight-bold">
                    <span style="color:red;">* </span>Tipo Producto
                  </span>:<br>
                  <span v-if="(tipoOperacion === 'Ver' || tipoOperacion === 'Eliminar')">
                    {{objeto.descripciontipoproducto}}
                  </span>
                  <b-form-select v-else v-model="objeto.idtipoproducto" class="mb-3">
                    <option :key="'tipo_' + t" v-for="(tipo, t) in tipoProductos" :value="tipo.id">{{tipo.descripcion}}</option>
                  </b-form-select>
                </b-col>
              </b-row>
            </b-container>
            <div slot="modal-footer" v-if="objeto !== null" class="pull-right">
              <b-btn
                v-if="tipoOperacion === 'Ver'"
                size="sm"
                variant="secondary"
                @click.stop="cerrarFormulario"
              >Cerrar</b-btn>
              <b-btn
                v-if="tipoOperacion === 'Agregar'"
                size="sm"
                variant="primary"
                @click.stop="guardar"
              >Guardar</b-btn>
              <b-btn
                v-if="tipoOperacion === 'Modificar'"
                size="sm"
                variant="primary"
                @click.stop="guardarCambios"
              >Guardar Cambios</b-btn>
              <b-btn
                v-if="tipoOperacion === 'Eliminar'"
                size="sm"
                variant="danger"
                @click.stop="eliminar"
              >Eliminar</b-btn>
            </div>
          </b-modal>
        </b-card>
      </b-col>
    </b-row>
  </div>
</template>
<style scoped>
.columna-centrada {
  text-align: center;
}
</style>

<script>
import {Money} from 'v-money'
var self = this;
export default {
  name: "Producto",
  data: function() {
    return {
      campos: [
        {
          key: "descripcion",
          label: "Descripción",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "descripciontipoproducto",
          label: "Tipo Producto",
          sortable: true,
          thStyle: "text-align:center;"
        },
        {
          key: "estado",
          label: "Estado",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "precio",
          label: "Precio",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "cantidad",
          label: "Cantidad",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "acciones",
          label: "Acciones",
          sortable: false,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        }
      ],
      items: [],
      objeto: null,
      showModal: false,
      tipoOperacion: "",
      totalFilas: 0,
      filtro: "",
      porPagina: 20,
      paginaActual: 1,
      estados: [],
      tipoProductos: []
    };
  },
  methods: {
    cargarFormulario: function(obj, operacion) {
      this.tipoOperacion = operacion;
      this.objeto = obj;
      if (obj === null) {
        this.objeto = {
          id: null,
          descripcion: null,
          orden: null,
          costo: '',
          precio: '',
          cantidad: null,
          estado: null,
          idtipoproducto: null,
          descripciontipoproducto: null
        };
      } else {
        this.selectedVisble = obj.visible
        this.objeto = {
          id: obj.id,
          descripcion: obj.descripcion,
          orden: obj.orden,
          costo: obj.costo,
          precio: obj.precio,
          cantidad: obj.cantidad,
          estado: obj.estado,
          idtipoproducto: obj.idtipoproducto,
          descripciontipoproducto: obj.descripciontipoproducto
        };
      }
      this.showModal = true;
    },
    validarCampos: function() {
      if (!this.objeto.descripcion || this.objeto.descripcion.trim().length < 1) {
        this.$toast.error("Debe escribir la descripción");
        return false;
      }
      if (!this.objeto.orden || this.objeto.orden.trim().length < 1) {
        this.$toast.error("Debe de escribir el orden");
        return false;
      }
      if (this.objeto.costo <= 0) {
        this.$toast.error("Debe escribir el costo");
        return false;
      }
      if (this.objeto.precio <= 0) {
        this.$toast.error("Debe escribir el precio");
        return false;
      }
      if (!this.objeto.cantidad || this.objeto.cantidad.trim().length < 1) {
        this.$toast.error("Debe escribir el cantidad");
        return false;
      }
      if (!this.objeto.estado || this.objeto.estado.trim().length < 1) {
        this.$toast.error("Debe seleccionar el estado");
        return false;
      }
      if (!this.objeto.idtipoproducto || this.objeto.idtipoproducto.trim().length < 1) {
        this.$toast.error("Debe seleccionar el tipo de producto");
        return false;
      }
      return true;
    },
    listarEstados: function() {
      this.$loader.open({ message: "Cargando ..." });
      var self = this;
      var frm = {};
      this.$http.get("ws/estado/", frm).then(resp => {
          self.estados = resp.data;
          self.$loader.close();
        }).catch(resp => {
          self.$loader.close();
          if (resp.data && resp.data.mensaje) {
            self.$toast.error(resp.data.mensaje);
          } else {
            self.$toast.error("No se pudo obtener los items de estados");
          }
        });
    },
    listarTipoProductos: function() {
      this.$loader.open({ message: "Cargando ..." });
      var self = this;
      var frm = {};
      this.$http.get("ws/tipoproducto/", frm).then(resp => {
          self.tipoProductos = resp.data;
          self.$loader.close();
        }).catch(resp => {
          self.$loader.close();
          if (resp.data && resp.data.mensaje) {
            self.$toast.error(resp.data.mensaje);
          } else {
            self.$toast.error("No se pudo obtener los items de tipos de productos");
          }
        });
    },
    listar: function() {
      this.$loader.open({ message: "Cargando ..." });
      var self = this;
      var frm = {};
      this.$http.get("ws/producto/", frm).then(resp => {
          self.items = resp.data;
          if (self.items && self.items.length > 0) {
            self.totalFilas = self.items.length;
          }
          self.$loader.close();
        })
        .catch(resp => {
          self.$loader.close();
          if (resp.data && resp.data.mensaje) {
            self.$toast.error(resp.data.mensaje);
          } else {
            self.$toast.error("No se pudo obtener los items");
          }
        });
    },
    guardar: function() {
      if(!this.validarCampos()) {
        return false
      }
      var self = this;
      self.$set(self.objeto, "token", window.localStorage.getItem("token"));
      self.objeto.visible = self.selectedVisble;
      this.$alertify
        .confirmWithTitle(
          "Guardar",
          "Seguro que desea guardar el nuevo registro?",
          function() {
            self.$loader.open({ message: "Guardando ..." });
            self.$http.post("ws/producto/", self.objeto).then(resp => {
                self.$loader.close();
                self.listar();
                self.$toast.success(resp.data.mensaje);
                self.showModal = false;
              })
              .catch(resp => {
                self.$loader.close();
                if (resp.data && resp.data.Error) {
                  self.$toast.error(resp.data.Error);
                } else {
                  self.$toast.error("error registrando");
                }
              });
          },
          function() {}
        )
        .set("labels", { ok: "Aceptar", cancel: "Cancelar" });
    },
    guardarCambios: function() {
      if(!this.validarCampos()) {
        return false
      }
      var self = this;
      self.$set(self.objeto, "token", window.localStorage.getItem("token"));
      self.objeto.visible = self.selectedVisble;
      this.$alertify
        .confirmWithTitle(
          "Modificar",
          "Seguro que desea modificar el registro?",
          function() {
            self.showModal = false;
            self.$loader.open({ message: "Guardando Cambios ..." });
            self.$http.put("ws/producto/", self.objeto).then(resp => {
                self.$loader.close();
                self.listar();
                self.$toast.success(resp.data.mensaje);
              })
              .catch(resp => {
                self.$loader.close();
                self.showModal = true;
                if (resp.data && resp.data.mensaje) {
                  self.$toast.error(resp.data.mensaje);
                } else {
                  self.$toast.error("error modificando");
                }
              });
          },
          function() {}
        )
        .set("labels", { ok: "Aceptar", cancel: "Cancelar" });
    },
    eliminar: function() {
      var self = this;
      let token = window.localStorage.getItem("token");
      self.$set(self.objeto, "token", window.localStorage.getItem("token"));
      this.$alertify
        .confirmWithTitle(
          "Eliminar",
          "Seguro que desea eliminar el registro?",
          function() {
            self.showModal = false;
            self.$loader.open({ message: "Eliminando ..." });
            self.$http.delete("ws/producto/", {params: {id: self.objeto.id, token: token}}).then(resp => {
                self.$loader.close();
                self.listar();
                self.$toast.success(resp.data.mensaje);
              })
              .catch(resp => {
                self.$loader.close();
                self.showModal = true;
                if (resp.data && resp.data.mensaje) {
                  self.$toast.error(resp.data.mensaje);
                } else {
                  self.$toast.error("error eliminando");
                }
              });
          },
          function() {}
        )
        .set("labels", { ok: "Aceptar", cancel: "Cancelar" });
    },
    cerrarFormulario: function() {
      this.showModal = false;
    },
    actualizarTabla: function(itemsFiltrados) {
      this.totalFilas = itemsFiltrados.length;
      this.paginaActual = 1;
    }
  },
  created: function() {
    this.listar();
    this.listarEstados();
    this.listarTipoProductos();
  },
  mounted: function() {
    this.$loader.close();
  }
};
</script>