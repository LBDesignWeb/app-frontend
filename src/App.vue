<script setup>
import { ref } from 'vue'

// Variable reactiva para el cuadro de texto
const entradaTexto = ref('')

// Función principal para conectar con el Backend
const guardarComo = async (tipo) => {
  // Validación inicial rápida en el Frontend
  if (!entradaTexto.value.trim()) {
    alert('Por favor, escribe o dicta una idea antes de guardar.')
    return
  }

  try {
    // Hacemos el puente usando fetch apuntando a tu API en XAMPP
    const respuesta = await fetch('http://localhost/api-agenda/guardar.php', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        contenido: entradaTexto.value,
        tipo_origen: tipo
      })
    })

    // Parseamos la respuesta JSON del servidor
    const datos = await respuesta.json()

    if (respuesta.ok && datos.success) {
      alert(`¡Éxito!: ${datos.mensaje}`)
      entradaTexto.value = '' // Limpiamos la caja de texto para la siguiente idea
    } else {
      alert(`Hubo un problema: ${datos.error || 'Error desconocido'}`)
    }

  } catch (error) {
    console.error('Error en la conexión:', error)
    alert('No se pudo conectar con el servidor. Asegúrate de que XAMPP (Apache/MySQL) esté corriendo.')
  }
}
</script>

<template>
  <main class="contenedor-app">
    <header class="encabezado">
      <h1>Hola Luis, ¿qué tenés en mente?</h1>
    </header>

    <section class="zona-captura">
      <textarea 
        v-model="entradaTexto"
        placeholder="Escribí o dictá tu idea acá sin vueltas..."
        rows="4"
        class="caja-entrada"
      ></textarea>
      
      <div class="bloque-botones">
        <button @click="guardarComo('idea')" class="btn btn-idea">
          <span>💡</span> Idea Rápida
        </button>
        <button @click="guardarComo('evento')" class="btn btn-evento">
          <span>📅</span> Evento Clave
        </button>
        <button @click="guardarComo('alarma')" class="btn btn-alarma">
          <span>⏰</span> Alarma Ya
        </button>
      </div>
    </section>
  </main>
</template>

<style scoped>
/* Estilos globales del contenedor móvil */
.contenedor-app {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  max-width: 480px;
  margin: 0 auto;
  padding: 20px;
  color: #333;
}

.encabezado h1 {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 25px;
  color: #1a1a1a;
}

/* Caja de texto Light & Clean */
.caja-entrada {
  width: 100%;
  box-sizing: border-box;
  padding: 15px;
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  background-color: #f9f9f9;
  font-size: 1rem;
  resize: none;
  outline: none;
  transition: all 0.3s ease;
  box-shadow: inset 0 1px 3px rgba(0,0,0,0.02);
}

.caja-entrada:focus {
  background-color: #ffffff;
  border-color: #a0a0a0;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}

/* Distribución de botones híbridos */
.bloque-botones {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin-top: 15px;
}

/* Estilo base de los botones móviles */
.btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 12px 8px;
  border: 1px solid #eaeaea;
  border-radius: 12px;
  background-color: #ffffff;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn span {
  font-size: 1.4rem;
  margin-bottom: 4px;
}

.btn:active {
  transform: scale(0.95);
}

/* Variaciones sutiles de color para cada intención */
.btn-idea:hover, .btn-idea:active {
  background-color: #f0f7ff;
  border-color: #bcd7ff;
}

.btn-evento:hover, .btn-evento:active {
  background-color: #fff9f0;
  border-color: #ffe3bc;
}

.btn-alarma:hover, .btn-alarma:active {
  background-color: #fff2f2;
  border-color: #ffcccc;
}
</style>