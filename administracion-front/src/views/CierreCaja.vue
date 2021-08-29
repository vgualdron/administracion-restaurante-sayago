<template>
  <div>
    <b-row>
      <b-col cols="12">
        <b-card border-variant="primary" header-bg-variant="primary" :header="'<strong>Cierre de caja</strong>'" tag="article" class="m-3 mt-3">
          <b-row class="mt-1 mb-2">
            <b-col cols="12">
              <b-form-group>
                <b-input-group cols="9">
                  <b-form-input
                    type="date"
                    v-model="filtro"
                    placeholder="Filtrar Búsqueda"
                    autocomplete="text"></b-form-input>
                  <!-- Attach Right button -->
                  <b-input-group-append>
                    <b-button variant="primary" :disabled="!filtro" @click.stop="filtro = ''">x</b-button>
                    <b-btn
                      class="ml-3"
                      @click.stop="consultarCierreCaja()"
                      variant="primary">
                      Generar Informe de cierre de caja</b-btn>
                  </b-input-group-append>
                </b-input-group>
              </b-form-group>
            </b-col>
          </b-row>

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
var self = this;
export default {
  name: "CierreCaja",
  data: function() {
    return {
      items: [],
      objeto: null,
      showModal: false,
      tipoOperacion: "",
      totalFilas: 0,
      filtro: "",
      porPagina: 20,
      paginaActual: 1
    };
  },
  methods: {
    consultarCierreCaja: function() {
      if (!this.filtro) {
        this.$toast.error("Debe de seleccionar la fecha para generar el cierre de caja");
        return false;
      }
      this.$loader.open({ message: "Cargando ..." });
      var self = this;
      var frm = {
        params: {
          fecha: this.filtro
        }
      };
      this.$http.get("ws/cierrecaja/", frm).then(resp => {
          self.items = resp.data;
          self.$loader.close();
          self.generarInformeCierreCaja();
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
    generarInformeCierreCaja: function() {
      if (!this.items || this.items.length <= 0) {
        this.$toast.error("No se encontraron ventas para hacer un cierre de caja cierre de caja");
        // return false;
      }
      var self = this;
      self.$loader.open({ message: "Generando ..." });
      var frm = {
        productos: this.items
      }
      /* self.$http.post("/dompdf/dompdf/www/cierre-caja.php", frm).then(resp => {
        self.$loader.close();
      })
      .catch(resp => {
        self.$loader.close();
        if (resp.data && resp.data.Error) {
          self.$toast.error(resp.data.Error);
        } else {
          self.$toast.error("error generando");
        }
      }); */
      var form = document.createElement("form");
      form.setAttribute("method", "get");
      form.setAttribute("action", process.env.VUE_APP_URL + "dompdf/dompdf/www/cierre-caja.php");

      form.setAttribute("target", "view");

      var hiddenField2 = document.createElement("input"); 
      hiddenField2.setAttribute("type", "hidden");
      hiddenField2.setAttribute("name", "fecha");
      hiddenField2.setAttribute("value", this.filtro);
      form.appendChild(hiddenField2);

      document.body.appendChild(form);

      window.open('', 'view');

      form.submit();
      self.$loader.close();
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
  },
  mounted: function() {
    this.$loader.close();
  }
};
</script>
