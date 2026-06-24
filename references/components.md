# Componentes Vuesax — Referencia Completa

## vs-table con paginación manual (server-side)

```vue
<vs-table
  v-model="selected"
  :data="registros"
  :no-data-text="'Sin registros'"
  multiple
  checkbox-color="primary"
>
  <template slot="header">
    <vs-input
      v-model="search"
      placeholder="Buscar..."
      icon-pack="feather"
      icon="icon-search"
      class="mb-4 w-64"
    />
  </template>

  <template slot="thead">
    <vs-th sort-key="nombre">Nombre</vs-th>
    <vs-th sort-key="email">Email</vs-th>
    <vs-th>Estado</vs-th>
    <vs-th>Acciones</vs-th>
  </template>

  <template slot-scope="{ data }">
    <vs-tr v-for="(row, i) in data" :key="i" :data="row">
      <vs-td :data="row.nombre">{{ row.nombre }}</vs-td>
      <vs-td :data="row.email">{{ row.email }}</vs-td>
      <vs-td>
        <vs-chip :color="getEstadoColor(row.estado)">
          {{ row.estado_label }}
        </vs-chip>
      </vs-td>
      <vs-td>
        <div class="flex gap-2">
          <vs-tooltip text="Ver detalle">
            <vs-button size="small" type="flat" color="primary"
              icon-pack="feather" icon="icon-eye" @click="show(row)" />
          </vs-tooltip>
          <vs-tooltip text="Editar">
            <vs-button size="small" type="border" color="warning"
              icon-pack="feather" icon="icon-edit" @click="edit(row)" />
          </vs-tooltip>
          <vs-tooltip text="Eliminar">
            <vs-button size="small" type="border" color="danger"
              icon-pack="feather" icon="icon-trash-2" @click="remove(row.id)" />
          </vs-tooltip>
        </div>
      </vs-td>
    </vs-tr>
  </template>
</vs-table>

<!-- Paginación manual -->
<div class="flex justify-end mt-4 px-6 pb-4">
  <vs-pagination
    v-model="currentPage"
    :total="totalPages"
    @input="fetchData"
  />
</div>
```

---

## vs-popup (Modal)

### Modal simple
```vue
<vs-popup title="Título del Modal" :active.sync="showPopup" class="popup-md">
  <div class="p-4">
    <!-- contenido -->
    <div class="flex justify-end gap-3 mt-4">
      <vs-button type="border" color="dark" @click="showPopup = false">
        Cancelar
      </vs-button>
      <vs-button color="primary" @click="confirm">Confirmar</vs-button>
    </div>
  </div>
</vs-popup>
```

### Tamaños de popup con CSS:
```css
/* En el componente padre o en el style scoped */
.popup-sm  .vs-popup { width: 400px  !important; }
.popup-md  .vs-popup { width: 600px  !important; }
.popup-lg  .vs-popup { width: 800px  !important; }
.popup-xl  .vs-popup { width: 1000px !important; }
```

---

## vs-input

```vue
<!-- Input básico -->
<vs-input
  v-model="form.nombre"
  label="Nombre completo"
  placeholder="Ej: Juan Pérez"
  class="w-full"
/>

<!-- Con ícono -->
<vs-input
  v-model="form.email"
  label="Correo electrónico"
  icon-pack="feather"
  icon="icon-mail"
  icon-after
  class="w-full"
/>

<!-- Con validación -->
<vs-input
  v-model="form.campo"
  label="Campo requerido"
  class="w-full"
  :danger="!!errors.campo"
  :danger-text="errors.campo ? errors.campo[0] : ''"
  :success="!errors.campo && form.campo.length > 0"
/>

<!-- Textarea -->
<vs-textarea
  v-model="form.descripcion"
  label="Descripción"
  rows="4"
  class="w-full"
/>
```

---

## vs-select

```vue
<!-- Select simple -->
<vs-select
  v-model="form.categoria_id"
  label="Categoría"
  class="w-full"
  autocomplete
>
  <vs-option
    v-for="cat in categorias"
    :key="cat.id"
    :label="cat.nombre"
    :value="cat.id"
  >
    {{ cat.nombre }}
  </vs-option>
</vs-select>

<!-- Select múltiple -->
<vs-select
  v-model="form.permisos"
  label="Permisos"
  class="w-full"
  multiple
>
  <vs-option
    v-for="p in permisos"
    :key="p.id"
    :label="p.nombre"
    :value="p.id"
  />
</vs-select>
```

---

## vs-card

```vue
<!-- Card estándar -->
<vx-card title="Título de la Card" subtitle="Subtítulo opcional">
  <p>Contenido aquí</p>

  <template slot="actions">
    <vs-button type="flat" color="primary" icon-pack="feather" icon="icon-settings" />
  </template>
</vx-card>

<!-- Card sin padding (para tablas) -->
<vx-card no-shadow card-border>
  <!-- tabla aquí -->
</vx-card>
```

---

## vs-alert

```vue
<!-- Inline en el template -->
<vs-alert
  v-if="alertMessage"
  :active.sync="showAlert"
  color="danger"
  icon-pack="feather"
  icon="icon-info"
  closable
>
  {{ alertMessage }}
</vs-alert>
```

---

## Loading con $vs.loading

```js
// Activar loading global
this.$vs.loading()

// Activar en contenedor específico
this.$vs.loading({ container: this.$refs.loadingContainer, scale: 0.6 })

// Desactivar
this.$vs.loading.close()
this.$vs.loading.close(this.$refs.loadingContainer.$el)
```

---

## Notificaciones ($vs.notify)

```js
// Éxito
this.$vs.notify({
  title:    'Éxito',
  text:     'Operación completada.',
  color:    'success',
  iconPack: 'feather',
  icon:     'icon-check-circle',
  position: 'top-right',
})

// Error
this.$vs.notify({
  title:    'Error',
  text:     'Algo salió mal. Intente nuevamente.',
  color:    'danger',
  iconPack: 'feather',
  icon:     'icon-alert-circle',
  position: 'top-right',
})

// Advertencia
this.$vs.notify({
  title:    'Atención',
  text:     'Por favor verifique los datos.',
  color:    'warning',
  iconPack: 'feather',
  icon:     'icon-alert-triangle',
})
```

---

## Dialog de Confirmación

```js
this.$vs.dialog({
  type:       'confirm',
  color:      'danger',
  title:      '¿Confirmar eliminación?',
  text:       'Esta acción no se puede deshacer.',
  accept:     this.deleteRecord,
  acceptText: 'Sí, eliminar',
  cancelText: 'Cancelar',
})
```

---

## Patrones de Layout

### Fila con filtros y botón
```vue
<div class="flex flex-wrap items-center justify-between gap-4 mb-6">
  <div class="flex flex-wrap gap-3">
    <vs-input v-model="search" placeholder="Buscar..." icon-pack="feather" icon="icon-search" class="w-64" />
    <vs-select v-model="filtro" placeholder="Filtrar por..." class="w-48">
      <vs-option value="">Todos</vs-option>
      <vs-option value="1">Activo</vs-option>
    </vs-select>
  </div>
  <vs-button color="primary" type="filled" icon-pack="feather" icon="icon-plus">
    Nuevo
  </vs-button>
</div>
```

### Grid de tarjetas de resumen (stats)
```vue
<div class="vx-row mb-6">
  <div v-for="stat in stats" :key="stat.label" class="vx-col w-full sm:w-1/2 lg:w-1/4 mb-4">
    <vx-card class="text-center">
      <feather-icon :icon="stat.icon" class="block mx-auto mb-2 text-primary" svgClasses="w-10 h-10" />
      <h2 class="font-bold text-2xl">{{ stat.value }}</h2>
      <p class="text-sm text-grey">{{ stat.label }}</p>
    </vx-card>
  </div>
</div>
```