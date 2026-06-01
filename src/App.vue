<script setup>
import { ref, onMounted } from 'vue'

// Variables reactivas
const entradaTexto = ref('')
const elementos = ref([]) // Acá guardaremos la lista para el barrido

// Función real para leer los datos desde la base de datos
const obtenerElementos = async () => {
  try {
    const respuesta = await fetch('http://localhost/api-agenda/listar.php')
    if (respuesta.ok) {
      const datos = await respuesta.json()
      elementos.value = datos // Reemplazamos los datos simulados por los reales de MySQL
    } else {
      console.error('Error al traer los elementos del servidor')
    }
  } catch (error) {
    console.error('Error de conexión al listar:', error)
  }
}

// Función principal para conectar con el Backend (Guardar)
const guardarComo = async (tipo) => {
  if (!entradaTexto.value.trim()) {
    alert('Por favor, escribe o dicta una idea antes de guardar.')
    return
  }

  try {
    const respuesta = await fetch('http://localhost/api-agenda/guardar.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        contenido: entradaTexto.value,
        tipo_origen: tipo
      })
    })

    const datos = await respuesta.json()

    if (respuesta.ok && datos.success) {
      alert(`¡Éxito!: ${datos.mensaje}`)
      entradaTexto.value = '' 
      obtenerElementos()
      // Luego de guardar, refrescaremos la lista automáticamente
    } else {
      alert(`Hubo un problema: ${datos.error || 'Error desconocido'}`)
    }
  } catch (error) {
    console.error('Error en la conexión:', error)
    alert('No se pudo conectar con el servidor.')
  }
}

// Al montar el componente, hacemos el barrido inicial de datos
onMounted(() => {
  obtenerElementos()
})
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

    <section class="seccion-lista">
      <h2>Notas Recientes</h2>
      
      <div v-if="elementos.length === 0" class="lista-vacia">
        No hay elementos registrados hoy.
      </div>

      <div v-else class="lista-elementos">
        <div 
          v-for="item in elementos" 
          :key="item.id_elemento" 
          :class="['tarjeta-item', `tarjeta-${item.tipo_origen}`]"
        >
          <div class="tarjeta-icono">
            <span v-if="item.tipo_origen === 'idea'">💡</span>
            <span v-if="item.tipo_origen === 'evento'">📅</span>
            <span v-if="item.tipo_origen === 'alarma'">⏰</span>
          </div>
          <div class="tarjeta-cuerpo">
            <p class="tarjeta-texto">{{ item.contenido }}</p>
            <span class="tarjeta-fecha">{{ item.fecha_creacion }}</span>
          </div>
        </div>
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
  text-align: center;
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

.btn-idea:hover, .btn-idea:active { background-color: #f0f7ff; border-color: #bcd7ff; }
.btn-evento:hover, .btn-evento:active { background-color: #fff9f0; border-color: #ffe3bc; }
.btn-alarma:hover, .btn-alarma:active { background-color: #fff2f2; border-color: #ffcccc; }

/* --- ESTILOS DE LA NUEVA SECCIÓN DE LISTADO --- */
.seccion-lista {
  margin-top: 40px;
}

.seccion-lista h2 {
  font-size: 1.1rem;
  font-weight: 600;
  color: #666;
  margin-bottom: 15px;
}

.lista-vacia {
  text-align: center;
  color: #999;
  font-size: 0.9rem;
  padding: 20px;
  border: 1px dashed #e0e0e0;
  border-radius: 12px;
}

.lista-elementos {
  display: flex;
  flex-direction: column; /* En celular van en lista vertical, súper práctico */
  gap: 12px;
  padding-bottom: 10px;
}

/* Tarjetas base con diseño limpio */
.tarjeta-item {
  display: flex;
  align-items: flex-start;
  padding: 14px;
  border-radius: 12px;
  border: 1px solid #f0f0f0;
  transition: transform 0.2s ease;
  width: 100%;
  box-sizing: border-box;
}

/* --- ADAPTACIÓN INTELIGENTE (Para Laptop / Pantallas Grandes) --- */
@media (min-width: 600px) {
  .lista-elementos {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* En la laptop se ponen los 3 en horizontal automáticos */
    gap: 12px;
  }
  
  .tarjeta-item {
    height: 100%; /* Hace que todas tengan la misma altura visual en la fila */
  }
}

.tarjeta-icono {
  font-size: 1.3rem;
  margin-right: 12px;
  margin-top: 2px;
}

.tarjeta-cuerpo {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.tarjeta-texto {
  font-size: 0.95rem;
  margin: 0 0 6px 0;
  line-height: 1.4;
  color: #2c2c2c;
  text-align: left;
}

.tarjeta-fecha {
  font-size: 0.75rem;
  color: #999;
  text-align: left;
}

/* Variaciones de fondo sutiles para el listado */
.tarjeta-idea { background-color: #f8fbff; border-color: #e6f0fa; }
.tarjeta-evento { background-color: #fffdf9; border-color: #fbf3e6; }
.tarjeta-alarma { background-color: #fffbfa; border-color: #fae8e6; }
</style>