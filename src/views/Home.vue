<template>
  <!-- Contenedor principal de la página -->
  <div class="container mx-auto py-8 px-4">
    
    <!-- Título de la sección -->
    <h1 class="text-4xl font-bold text-center text-teal-700 mb-8 transition duration-500 ease-in-out transform hover:scale-105">
      Leyendas Costarricenses
    </h1>

    <!-- Campo de búsqueda para filtrar leyendas -->
    <div class="mb-6">
      <input 
        v-model="searchQuery" 
        type="text" 
        class="w-full p-3 border border-gray-300 rounded-lg" 
        placeholder="Buscar por nombre, categoría, provincia, cantón, distrito..." 
      />
    </div>

    <!-- Botón para agregar una nueva leyenda -->
    <div class="mb-6">
      <router-link to="/create">
        <button class="px-6 py-3 bg-teal-500 text-white font-semibold rounded-lg shadow-md hover:bg-teal-600 transition duration-300">
          Agregar Leyenda
        </button>
      </router-link>
    </div>

    <!-- Lista de leyendas filtradas según la búsqueda -->
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-8">
      <div v-for="leyenda in filteredLeyendas" :key="leyenda.id" class="transform transition-all hover:scale-105 hover:shadow-2xl rounded-lg overflow-hidden bg-white shadow-md hover:shadow-lg duration-300 ease-in-out">
        <!-- Imagen de la leyenda -->
        <img :src="leyenda.image_url" alt="Leyenda" class="w-full h-56 object-cover transition-transform duration-500 transform hover:scale-110"/>
        
        <!-- Descripción de la leyenda -->
        <div class="p-6">
          <!-- Nombre de la leyenda -->
          <h2 class="text-2xl font-semibold text-gray-800 mb-2 transition duration-300 ease-in-out transform hover:translate-x-2">
            {{ leyenda.name }}
          </h2>
          <p class="text-sm text-gray-600 mb-3">{{ leyenda.category }}</p>
          <p class="text-gray-700 mt-2 mb-4">{{ leyenda.description }}</p>
          
          <!-- Detalles adicionales sobre la leyenda -->
          <div class="flex flex-col space-y-1 text-sm text-gray-500">
            <p><strong>Provincia:</strong> {{ leyenda.province }}</p>
            <p><strong>Cantón:</strong> {{ leyenda.canton }}</p>
            <p><strong>Distrito:</strong> {{ leyenda.district }}</p>
            <p>{{ formatRelativeDate(leyenda.created_at) }}</p>

            <!-- Contenedor de los botones de editar y eliminar -->
            <div class="flex justify-between mt-6">
              <!-- Botón de Editar -->
              <router-link :to="`/edit/${leyenda.id}`">
                <button class="px-5 py-2 bg-teal-500 text-white font-semibold rounded-lg shadow-md hover:bg-teal-600 transition duration-300">
                  Editar
                </button>
              </router-link>

              <!-- Botón de Eliminar -->
              <button @click="confirmDelete(leyenda.id)" class="px-5 py-2 bg-red-500 text-white font-semibold rounded-lg shadow-md hover:bg-red-600 transition duration-300">
                Eliminar
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Mensajes de estado: éxito o error -->
    <div v-if="message" class="fixed bottom-5 left-5 bg-teal-600 text-white px-4 py-2 rounded-lg shadow-lg">
      {{ message }}
    </div>
    <div v-if="errorMessage" class="fixed bottom-5 left-5 bg-red-600 text-white px-4 py-2 rounded-lg shadow-lg">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script>
// Importaciones necesarias
import { ref, onMounted, computed } from 'vue';
import { getLegends } from '../services/api';  // Función para obtener leyendas desde el backend
import { formatDistanceToNow } from 'date-fns';  // Librería para manejar fechas
import { deleteLegend } from '../services/api'; // Función para eliminar leyenda
import Swal from 'sweetalert2';  // Librería para mostrar alertas
import { es } from 'date-fns/locale';  // Localización para español

export default {
  name: 'LeyendasList',
  setup() {
    // Variables reactivas para el estado del componente
    const leyendas = ref([]);  // Almacena las leyendas obtenidas
    const error = ref(null);  // Almacena posibles errores
    const searchQuery = ref('');  // Filtro de búsqueda
    const message = ref(null);  // Mensaje de éxito
    const errorMessage = ref(null);  // Mensaje de error

    // Función para obtener leyendas desde el API
    const fetchLeyendas = async () => {
      try {
        const response = await getLegends();  // Llamada al API
        leyendas.value = response.data;  // Asignamos los datos obtenidos
      } catch (err) {
        errorMessage.value = 'Hubo un error al cargar las leyendas';  // Manejamos el error
        console.error('Error al cargar leyendas:', err);
      }
    };

    // Función para formatear la fecha de creación de las leyendas
    const formatRelativeDate = (dateString) => {
      const date = new Date(dateString);  // Convertimos la fecha a un objeto Date
      return formatDistanceToNow(date, { addSuffix: true, locale: es }); // Mostramos la distancia relativa de la fecha
    };

    // Computed property para filtrar leyendas según el término de búsqueda
    const filteredLeyendas = computed(() => {
      if (!searchQuery.value) return leyendas.value;  // Si no hay filtro, retornamos todas las leyendas
      return leyendas.value.filter((leyenda) => {
        const query = searchQuery.value.toLowerCase();  // Convertimos la búsqueda a minúsculas
        return (
          leyenda.name.toLowerCase().includes(query) ||
          leyenda.category.toLowerCase().includes(query) ||
          leyenda.province.toLowerCase().includes(query) ||
          leyenda.canton.toLowerCase().includes(query) ||
          leyenda.district.toLowerCase().includes(query) ||
          formatRelativeDate(leyenda.created_at).toLowerCase().includes(query) // También filtramos por fecha
        );
      });
    });

    // Función para confirmar la eliminación de una leyenda
    const confirmDelete = (id) => {
      Swal.fire({
        title: '¿Estás seguro?',
        text: 'Esta leyenda será eliminada permanentemente.',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Sí, eliminar',
        cancelButtonText: 'Cancelar',
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
      }).then((result) => {
        if (result.isConfirmed) {
          deleteConfirmed(id);  // Si el usuario confirma, eliminamos la leyenda
        }
      });
    };

    // Función para eliminar una leyenda después de la confirmación
    const deleteConfirmed = async (id) => {
      try {
        await deleteLegend(id);  // Llamamos al API para eliminar la leyenda
        leyendas.value = leyendas.value.filter(leyenda => leyenda.id !== id);  // Eliminamos de la lista local
        message.value = 'Leyenda eliminada con éxito';  // Mensaje de éxito
      } catch (err) {
        errorMessage.value = 'Hubo un error al eliminar la leyenda';  // Manejamos el error
        console.error('Error al eliminar leyenda:', err);
      }
    };

    // Cargamos las leyendas al montar el componente
    onMounted(() => {
      fetchLeyendas();
    });

    return {
      leyendas,
      error,
      searchQuery,
      filteredLeyendas,
      formatRelativeDate,
      confirmDelete,
      message,
      errorMessage
    };
  }
};
</script>

<style scoped>
/* Estilos generales para los botones */
button {
  transition: background-color 0.3s ease, transform 0.3s ease;
}

/* Animación de fade-in para el contenedor */
@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

.container {
  animation: fadeIn 1s ease-in-out;  /* Aplicamos la animación de fade-in */
}
</style>
