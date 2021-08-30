<template>
  <div>
    <b-row>
      <b-col cols="12">
        <b-card border-variant="primary" header-bg-variant="primary" :header="'<strong>Generar Facturas</strong> (' + totalFilas + ')'" tag="article" class="m-3 mt-3">
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
                class="tabla columna-centrada"
                :fields="campos"
                :items="items"
                :current-page="paginaActual"
                :per-page="porPagina"
                :filter="filtro"
                @filtered="actualizarTabla">
                <template slot="acciones" slot-scope="row">
                  <b-button
                    style="margin: 1px;"
                    size="sm"
                    @click.stop="cargarFormulario(row.item,'Ver')">
                    <i class="icon-eye"></i>
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

          <b-modal v-if="objeto" centered v-model="showModal" size="lg" title="Detalle de la Factura">
            <b-container>
              <b-row class="mb-3">
                <b-col class="text-left">
                  <b-form-group
                    id="nombreCliente"
                    description="Solo si es necesario."
                    label="<strong>Escriba el Nombre del Cliente</strong>">
                    <b-form-input id="input-cliente" v-model="nombreCliente" trim></b-form-input>
                  </b-form-group>
                  <b-table
                    v-if="itemsFactura && itemsFactura.length > 0"
                    stacked="md"
                    striped
                    hover
                    class="tabla columna-centrada"
                    :fields="camposFactura"
                    :items="itemsFactura">
                  </b-table>
                  <b-row v-if="itemsFactura && itemsFactura.length > 0" class="mt-1 mb-2 text-center">
                    <b-col cols="12">
                      <b-alert show><strong>TOTAL: {{totalPedido()}}</strong></b-alert>
                    </b-col>
                  </b-row>
                  <b-alert v-else show>No se encontraron registros.</b-alert>
                </b-col>
              </b-row>
            </b-container>
            <div slot="modal-footer" v-if="objeto !== null" class="pull-right">
              <b-btn
                variant="primary"
                @click.stop="generarTicketFactura">Generar Factura</b-btn>
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
  name: "Factura",
  data: function() {
    return {
      campos: [
        {
          key: "id",
          label: "Referencia",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "fechacambio",
          label: "Fecha",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "mesa",
          label: "Mesa",
          sortable: true,
          thStyle: "text-align:center;"
        },
        {
          key: "mesero",
          label: "Mesero",
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
      camposFactura: [
        {
          key: "descripcionproducto",
          label: "Producto",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "cantidadproducto",
          label: "Cantidad",
          sortable: true,
          thStyle: "text-align:center;"
        },
        {
          key: "precioproducto",
          label: "Precio",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        },
        {
          key: "subtotal",
          label: "Subtotal",
          sortable: true,
          thStyle: "text-align:center;",
          tdClass: "columna-centrada"
        }
      ],
      itemsFactura: [],
      objeto: null,
      showModal: false,
      tipoOperacion: "",
      totalFilas: 0,
      filtro: "",
      porPagina: 20,
      paginaActual: 1,
      nombreCliente: ''
    };
  },
  methods: {
    cargarFormulario: function(obj, operacion) {
      this.tipoOperacion = operacion;
      this.objeto = obj;
      this.consultarDetalleFactura(obj.id)
      this.showModal = true;
    },
    listar: function() {
      this.$loader.open({ message: "Cargando ..." });
      var self = this;
      var frm = {};
      this.$http.get("ws/factura/", frm).then(resp => {
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
    consultarDetalleFactura: function(idPedido) {
      this.$loader.open({ message: "Cargando ..." });
      var self = this;
      var frm = {
        params: {
          idPedido: idPedido
        }
      };
      this.$http.get("ws/detallefactura/", frm).then(resp => {
          self.itemsFactura = resp.data;
          self.$loader.close();
        })
        .catch(resp => {
          self.$loader.close();
          if (resp.data && resp.data.mensaje) {
            self.$toast.error(resp.data.mensaje);
          } else {
            self.$toast.error("No se pudo obtener los items de detalles de factura");
          }
        });
    },
    guardar: function() {
      var self = this;
      self.$set(self.objeto, "token", window.localStorage.getItem("token"));
      self.objeto.visible = self.selectedVisble;
      this.$alertify
        .confirmWithTitle(
          "Guardar",
          "Seguro que desea guardar el nuevo registro?",
          function() {
            self.$loader.open({ message: "Guardando ..." });
            self.$http.post("ws/gasto/", self.objeto).then(resp => {
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
    cerrarFormulario: function() {
      this.showModal = false;
    },
    actualizarTabla: function(itemsFiltrados) {
      this.totalFilas = itemsFiltrados.length;
      this.paginaActual = 1;
    },
    totalPedido: function() {
      var self = this;
      var total = 0;
      this.itemsFactura.forEach(item => {
        total += (parseInt(item.precioproducto) * parseInt(item.cantidadproducto));
      });
      var total = '$' + Number(total.toFixed(1)).toLocaleString();
      return total;
    },
    generarTicketFactura: function() {
      var self = this;
      var frm = { 
        productos: self.itemsFactura,
        mesa: self.objeto.mesa,
        mesero: self.objeto.mesero,
        cliente: self.nombreCliente
      }
      self.$loader.open({ message: "Generando ..." });
      self.$http.post("ws/ticket/factura.php", frm).then(resp => {
          self.$loader.close();
          // self.$toast.success(resp.data.mensaje);
          // self.$toast.success("Exito");
        })
        .catch(resp => {
          self.$loader.close();
          self.$toast.error("error generando ticket de factura");
        });
    },
  },
  created: function() {
    this.listar();
  },
  mounted: function() {
    this.$loader.close();
  }
};
</script>
