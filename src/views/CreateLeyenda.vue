<template>
  <div class="container mx-auto py-8 px-4">
    <!-- Título de la página -->
    <h1 class="text-4xl font-bold text-center text-teal-700 mb-8">
      Crear Nueva Leyenda
    </h1>

    <!-- Formulario para crear leyenda -->
    <form @submit.prevent="submitForm" class="space-y-6 max-w-4xl mx-auto">
      
      <!-- Campo para el nombre de la leyenda -->
      <div>
        <label for="name" class="block text-lg font-semibold">Nombre de la Leyenda</label>
        <input v-model="leyenda.name" type="text" id="name" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Campo para la categoría de la leyenda -->
      <div>
        <label for="category" class="block text-lg font-semibold">Categoría</label>
        <input v-model="leyenda.category" type="text" id="category" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Campo para la descripción de la leyenda -->
      <div>
        <label for="description" class="block text-lg font-semibold">Descripción</label>
        <textarea v-model="leyenda.description" id="description" rows="4" class="w-full p-3 border border-gray-300 rounded-lg" required></textarea>
      </div>

      <!-- Campo para la provincia -->
      <div>
        <label for="province" class="block text-lg font-semibold">Provincia</label>
        <input v-model="leyenda.province" type="text" id="province" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Campo para el cantón -->
      <div>
        <label for="canton" class="block text-lg font-semibold">Cantón</label>
        <input v-model="leyenda.canton" type="text" id="canton" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Campo para el distrito -->
      <div>
        <label for="district" class="block text-lg font-semibold">Distrito</label>
        <input v-model="leyenda.district" type="text" id="district" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Campo para la URL de la imagen -->
      <div>
        <label for="image_url" class="block text-lg font-semibold">URL de la Imagen</label>
        <input v-model="leyenda.image_url" type="url" id="image_url" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Vista previa de la imagen si existe una URL -->
      <div v-if="leyenda.image_url" class="mt-4">
        <p class="text-lg font-semibold">Vista previa de la Imagen:</p>
        <img :src="leyenda.image_url" alt="Vista previa de la imagen" class="w-full h-auto mt-2 border rounded-lg shadow-lg" />
      </div>

      <!-- Campo para la fecha de creación -->
      <div>
        <label for="created_at" class="block text-lg font-semibold">Fecha de Creación</label>
        <input v-model="leyenda.created_at" type="datetime-local" id="created_at" class="w-full p-3 border border-gray-300 rounded-lg" required />
      </div>

      <!-- Botones de acción -->
      <div class="flex justify-between space-x-4 mt-6">
        <!-- Botón para crear la leyenda -->
        <div class="w-1/2">
          <button type="submit" class="w-full px-6 py-3 bg-teal-500 text-white font-semibold rounded-lg shadow-md hover:bg-teal-600 transition duration-300">
            Crear Leyenda
          </button>
        </div>

        <!-- Botón para volver a la página anterior -->
        <div class="w-1/2">
          <button @click="goBack" type="button" class="w-full px-6 py-3 bg-gray-500 text-white font-semibold rounded-lg shadow-md hover:bg-gray-600 transition duration-300">
            Volver
          </button>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
// Importación de dependencias necesarias
import { ref } from 'vue'; // Ref para la reactividad
import { createLegend } from '../services/api'; // Función para crear la leyenda en el backend
import { useRouter } from 'vue-router'; // Para navegar entre páginas
import Swal from 'sweetalert2';  // Para mostrar alertas de éxito o error

export default {
  name: 'createLegend',  // Nombre del componente
  setup() {
    // Definición del objeto 'leyenda' que mantiene los datos del formulario
    const leyenda = ref({
      name: '', // Nombre de la leyenda
      category: '', // Categoría de la leyenda
      description: '', // Descripción de la leyenda
      province: '', // Provincia de la leyenda
      canton: '', // Cantón de la leyenda
      district: '', // Distrito de la leyenda
      image_url: '', // URL de la imagen
      created_at: new Date().toISOString()  // Fecha de creación de la leyenda (inicializada con la fecha actual)
    });

    // Uso del router para redirigir a la página principal
    const router = useRouter();  

    // Función para manejar el envío del formulario
    const submitForm = async () => {
      try {
        // Llamada al servicio de backend para crear la leyenda
        await createLegend(leyenda.value);
        
        // Mostrar alerta de éxito utilizando SweetAlert2
        Swal.fire({
          title: 'Éxito!',
          text: 'Leyenda creada con éxito.',
          icon: 'success',
          confirmButtonText: 'Aceptar'
        });

        // Limpiar el formulario después de enviar los datos
        leyenda.value = {  
          name: '',
          category: '',
          description: '',
          province: '',
          canton: '',
          district: '',
          image_url: '',
          created_at: new Date().toISOString()  
        };

        // Redirigir a la página principal después de crear la leyenda
        router.push('/');  

      } catch (err) {
        // En caso de error, mostrar una alerta de error
        Swal.fire({
          title: 'Error!',
          text: 'Hubo un error al crear la leyenda.',
          icon: 'error',
          confirmButtonText: 'Aceptar'
        });
        console.error(err); // Mostrar el error en la consola para depuración
      }
    };

    // Función para volver a la página anterior
    const goBack = () => {
      router.go(-1);  
    };

    // Retornar los datos y funciones que estarán disponibles en la plantilla
    return {
      leyenda,        // El objeto 'leyenda' que mantiene los datos del formulario
      submitForm,     // La función que maneja el envío del formulario
      goBack          // La función que maneja la navegación hacia atrás
    };
  }
};
</script>

<style scoped>
/* Estilo general para los botones */
button {
  transition: background-color 0.3s ease; /* Transición suave para el color de fondo */
}

button:hover {
  background-color: #2b6cb0; /* Color de fondo cuando el cursor pasa sobre el botón */
}
</style>
