<template>
    <!-- Contenedor principal con el formulario para editar una leyenda -->
    <div class="max-w-3xl mx-auto py-10 px-6 bg-white shadow-lg rounded-lg">
      <h1 class="text-3xl font-bold text-center text-teal-700 mb-6">Editar Leyenda</h1>
  
      <!-- Mensajes de estado de carga o error -->
      <div v-if="loading" class="text-center text-gray-600">Cargando...</div>
      <div v-if="error" class="text-red-500 text-center font-semibold">{{ error }}</div>
  
      <!-- Formulario para editar los datos de la leyenda -->
      <form v-if="leyendaData" @submit.prevent="editarLeyenda" class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <!-- Campos del formulario con v-model para enlazar los datos -->
        <div>
          <label class="block text-gray-700 font-semibold">Nombre</label>
          <input type="text" v-model="leyendaData.name" class="input-field" placeholder="Ingrese el nombre" />
        </div>
  
        <div>
          <label class="block text-gray-700 font-semibold">Categoría</label>
          <input type="text" v-model="leyendaData.category" class="input-field" placeholder="Ingrese la categoría" />
        </div>
  
        <div>
          <label class="block text-gray-700 font-semibold">Provincia</label>
          <input type="text" v-model="leyendaData.province" class="input-field" placeholder="Ingrese la provincia" />
        </div>
  
        <div>
          <label class="block text-gray-700 font-semibold">Cantón</label>
          <input type="text" v-model="leyendaData.canton" class="input-field" placeholder="Ingrese el cantón" />
        </div>
  
        <div>
          <label class="block text-gray-700 font-semibold">Distrito</label>
          <input type="text" v-model="leyendaData.district" class="input-field" placeholder="Ingrese el distrito" />
        </div>
  
        <!-- Vista previa de la imagen -->
        <div>
          <label class="block text-gray-700 font-semibold">Imagen (URL)</label>
          <input type="text" v-model="leyendaData.image_url" class="input-field" placeholder="Ingrese la URL de la imagen" />
          <div v-if="previewImage" class="mt-3">
            <img :src="previewImage" alt="Vista previa" class="rounded-md w-full h-48 object-cover shadow-md border" />
          </div>
        </div>
  
        <!-- Campo de descripción -->
        <div class="md:col-span-2">
          <label class="block text-gray-700 font-semibold">Descripción</label>
          <textarea v-model="leyendaData.description" class="input-field h-24" placeholder="Ingrese la descripción"></textarea>
        </div>
  
        <!-- Campo de fecha de creación -->
        <div>
          <label class="block text-gray-700 font-semibold">Fecha de Creación</label>
          <input type="datetime-local" v-model="leyendaData.created_at" class="input-field" />
        </div>
  
        <!-- Botones de acción -->
        <div class="md:col-span-2 flex justify-center gap-4 mt-4">
          <button type="submit" class="btn-primary">
            <i class="fas fa-save mr-2"></i> Guardar Cambios
          </button>
          <button type="button" @click="confirmarCancelacion" class="btn-secondary">
            <i class="fas fa-times mr-2"></i> Cancelar
          </button>
        </div>
      </form>
    </div>
  </template>
  
  <script>
  import { getLegendById, updateLegend } from "../services/api";
  import Swal from "sweetalert2";
  
  export default {
    name: "LeyendaEdit", // Nombre del componente.
    props: ["id"], // Propiedad para recibir el ID de la leyenda a editar.
    data() {
      return {
        leyendaId: this.id, // ID de la leyenda.
        leyendaData: null, // Datos de la leyenda que se van a editar.
        loading: true, // Indicador de carga.
        error: null, // Mensaje de error si ocurre algún problema.
      };
    },
    async mounted() {
      try {
        // Cargar los datos de la leyenda desde la API.
        const response = await getLegendById(this.leyendaId);
        this.leyendaData = response.data;
      } catch (err) {
        console.error(err);
        this.error = "No se pudo cargar la leyenda"; // Mostrar mensaje de error si la leyenda no se carga.
      } finally {
        this.loading = false; // Finalizar la carga.
      }
    },
    computed: {
      // Vista previa de la imagen si la URL es válida.
      previewImage() {
        return this.leyendaData?.image_url || null;
      },
    },
    methods: {
      // Método para editar los datos de la leyenda
      async editarLeyenda() {
        this.error = null; // Limpiar errores previos
        try {
          await updateLegend(this.leyendaId, this.leyendaData); // Enviar los datos a la API para actualización.
          Swal.fire({
            title: "¡Éxito!",
            text: "La leyenda ha sido actualizada con éxito.",
            icon: "success",
            confirmButtonText: "Aceptar",
          }).then(() => {
            this.$router.push("/"); // Redirigir al usuario a la página principal después de la actualización.
          });
        } catch (err) {
          console.error(err);
          this.error = "Hubo un problema al actualizar la leyenda"; // Mostrar error si la actualización falla.
        }
      },
      // Confirmación antes de cancelar la edición
      confirmarCancelacion() {
        Swal.fire({
          title: "¿Estás seguro?",
          text: "Perderás los cambios si cancelas la edición.",
          icon: "warning",
          showCancelButton: true,
          confirmButtonText: "Sí, cancelar",
          cancelButtonText: "No, continuar editando",
          reverseButtons: true,
        }).then((result) => {
          if (result.isConfirmed) {
            this.$router.push("/"); // Redirigir a la página principal si se confirma la cancelación.
          }
        });
      },
    },
  };
  </script>
  
  <style scoped>
  .input-field {
    @apply w-full px-4 py-2 mt-1 border rounded-md focus:ring-2 focus:ring-teal-500 focus:outline-none transition;
  }
  .btn-primary {
    @apply px-6 py-2 bg-teal-500 text-white font-semibold rounded-lg shadow-md hover:bg-teal-600 transition duration-300 flex items-center;
  }
  .btn-secondary {
    @apply px-6 py-2 bg-gray-500 text-white font-semibold rounded-lg shadow-md hover:bg-gray-600 transition duration-300 flex items-center;
  }
  </style>
  