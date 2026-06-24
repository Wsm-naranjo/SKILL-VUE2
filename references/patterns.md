# Patrones Avanzados — Laravel + Vue 2 + Vuexy

## Exportar a Excel desde Laravel

```php
// Controller
public function export(Request $request)
{
    $data = NombreModelo::query()
        ->when($request->search, fn($q, $s) => $q->where('nombre', 'like', "%$s%"))
        ->get();

    $headers = [
        'Content-Type' => 'application/vnd.ms-excel',
        'Content-Disposition' => 'attachment; filename="reporte.csv"',
    ];

    $csv = "Campo1,Campo2,Estado\n";
    foreach ($data as $row) {
        $csv .= "{$row->campo1},{$row->campo2},{$row->activo}\n";
    }

    return response($csv, 200, $headers);
}
```

```js
// Vue — descargar
async exportar() {
  const { data } = await axios.get('/api/nombre-modulo/export', {
    params: this.filters,
    responseType: 'blob',
  })
  const url  = window.URL.createObjectURL(new Blob([data]))
  const link = document.createElement('a')
  link.href  = url
  link.setAttribute('download', 'reporte.csv')
  document.body.appendChild(link)
  link.click()
  link.remove()
}
```

---

## Upload de Archivos

```vue
<!-- Template -->
<div class="con-img-upload">
  <input
    type="file"
    ref="fileInput"
    accept=".pdf,.jpg,.png"
    style="display:none"
    @change="handleFile"
  />
  <vs-button
    type="border"
    color="primary"
    icon-pack="feather"
    icon="icon-upload"
    @click="$refs.fileInput.click()"
  >
    Subir Archivo
  </vs-button>
  <span v-if="form.archivo" class="ml-3 text-sm text-grey">
    {{ form.archivo.name }}
  </span>
</div>
```

```js
// Methods
handleFile(e) {
  this.form.archivo = e.target.files[0]
},

async save() {
  const fd = new FormData()
  Object.keys(this.form).forEach(k => fd.append(k, this.form[k]))

  await axios.post('/api/ruta', fd, {
    headers: { 'Content-Type': 'multipart/form-data' }
  })
}
```

---

## Búsqueda con Debounce

```js
import { debounce } from 'lodash'

// En el componente
watch: {
  searchQuery: debounce(function() {
    this.currentPage = 1
    this.fetchData()
  }, 400),
},
```

---

## Mixin reutilizable para CRUDs

```js
// src/mixins/crudMixin.js
export default {
  data() {
    return {
      registros:   [],
      total:       0,
      currentPage: 1,
      perPage:     15,
      loading:     false,
      saving:      false,
      showPopup:   false,
      isEditing:   false,
      errors:      {},
    }
  },

  computed: {
    totalPages() {
      return Math.ceil(this.total / this.perPage)
    },
  },

  methods: {
    notify(color, text) {
      this.$vs.notify({
        title:    color === 'success' ? 'Éxito' : 'Error',
        text,
        color,
        iconPack: 'feather',
        icon:     color === 'success' ? 'icon-check-circle' : 'icon-alert-circle',
        position: 'top-right',
      })
    },

    confirmDelete(id, callback) {
      this.$vs.dialog({
        type:       'confirm',
        color:      'danger',
        title:      '¿Confirmar eliminación?',
        text:       'Esta acción no se puede deshacer.',
        accept:     () => callback(id),
        acceptText: 'Sí, eliminar',
        cancelText: 'Cancelar',
      })
    },

    handleErrors(e) {
      if (e.response?.status === 422) {
        this.errors = e.response.data.errors || {}
        this.notify('warning', 'Verifica los campos requeridos.')
      } else {
        this.notify('danger', 'Ocurrió un error inesperado.')
      }
    },
  },
}
```

---

## Scopes en Laravel (sin migraciones)

```php
// En el modelo
public function scopeActivos($query)
{
    return $query->where('activo', 1);
}

public function scopeBuscar($query, $termino)
{
    return $query->where(function($q) use ($termino) {
        $q->where('nombre', 'like', "%$termino%")
          ->orWhere('descripcion', 'like', "%$termino%");
    });
}

// Uso en controller
NombreModelo::activos()->buscar($request->search)->paginate(15);
```

---

## Route Model Binding (sin migración)

```php
// Si el PK no es 'id':
// En el modelo
public function getRouteKeyName()
{
    return 'codigo'; // o el campo que corresponda
}
```

---

## Accessor / Mutator (Laravel)

```php
// Accessor: campo_label automático
public function getEstadoLabelAttribute()
{
    return match($this->estado) {
        1 => 'Activo',
        2 => 'Pendiente',
        0 => 'Inactivo',
        default => 'Desconocido',
    };
}

// Append al JSON
protected $appends = ['estado_label'];
```

---

## Vuex Module para un CRUD

```js
// store/modules/nombreModulo.js
import NombreModuloApi from '@/api/nombreModulo'

export default {
  namespaced: true,

  state: {
    registros: [],
    total:     0,
  },

  getters: {
    all:   s => s.registros,
    total: s => s.total,
  },

  mutations: {
    SET_DATA(state, { data, total }) {
      state.registros = data
      state.total     = total
    },
  },

  actions: {
    async fetch({ commit }, params) {
      const { data } = await NombreModuloApi.getAll(params)
      commit('SET_DATA', { data: data.data, total: data.total })
    },
  },
}
```