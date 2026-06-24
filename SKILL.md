---
name: laravel-vue2-vuexy
description: >
  Use this skill for ALL development tasks involving Laravel (backend) and Vue 2 with the Vuexy template (frontend).
  Trigger whenever the user mentions: Laravel routes, controllers, models, APIs, Vue components, Vuexy, vs-table,
  vs-popup, vs-select, vs-input, vs-button, vs-chip, vs-card, Vuesax, CRUD, forms, modals, tables, or any
  full-stack feature in this stack. This skill enforces professional, elegant code standards — never generate
  Laravel migrations, never use raw HTML tables, never use Bootstrap or Vuetify components. Always use Vuesax
  components and Vuexy design patterns. Apply this skill even for small tasks like "add a column to the table"
  or "create a new route" — consistency matters.
---

# Laravel + Vue 2 + Vuexy Development Skill

## Stack Overview

| Layer | Technology |
|---|---|
| Backend | Laravel (sin migraciones — BD ya existe) |
| Frontend | Vue 2 + Vuexy Template |
| UI Library | Vuesax (vs-* components) |
| HTTP Client | Axios |
| State | Vuex (si aplica) |
| Routing | Vue Router |

---

## REGLAS ABSOLUTAS

1. **NUNCA generar migraciones de base de datos** — la BD ya existe. Solo usar modelos Eloquent con `$table`, `$fillable`, `$primaryKey` según corresponda.
2. **NUNCA usar Bootstrap, Vuetify, Element UI ni HTML puro** para UI. Solo componentes Vuesax (`vs-*`).
3. **NUNCA usar `<table>` HTML crudo** — siempre `vs-table` con `vs-th`, `vs-tr`, `vs-td`.
4. **SIEMPRE seguir la estructura de archivos de Vuexy** (ver sección de estructura).
5. **SIEMPRE usar estilos profesionales** — sin colores hardcodeados, usar variables CSS de Vuexy o clases de Vuexy.

---

## Backend — Laravel (Sin Migraciones)

### Estructura de un Modelo
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class NombreModelo extends Model
{
    protected $table = 'nombre_tabla';      // siempre declarar
    protected $primaryKey = 'id';           // si difiere del default
    public $timestamps = true;              // false si la tabla no tiene created_at/updated_at

    protected $fillable = [
        'campo1',
        'campo2',
        'campo3',
    ];

    protected $casts = [
        'fecha_campo' => 'date',
        'activo'      => 'boolean',
    ];

    // Relaciones
    public function relacion()
    {
        return $this->belongsTo(OtroModelo::class, 'fk_campo');
    }
}
```

### Estructura de un Controller (API Resource)
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\NombreModelo;
use Illuminate\Http\Request;

class NombreController extends Controller
{
    public function index(Request $request)
    {
        $data = NombreModelo::query()
            ->when($request->search, fn($q, $s) => $q->where('nombre', 'like', "%$s%"))
            ->orderBy('created_at', 'desc')
            ->paginate($request->per_page ?? 15);

        return response()->json([
            'success' => true,
            'data'    => $data->items(),
            'total'   => $data->total(),
        ]);
    }

    public function store(Request $request)
    {
        $request->validate([
            'campo1' => 'required|string|max:255',
        ]);

        $registro = NombreModelo::create($request->validated());

        return response()->json([
            'success' => true,
            'data'    => $registro,
            'message' => 'Registro creado correctamente.',
        ], 201);
    }

    public function show($id)
    {
        $registro = NombreModelo::findOrFail($id);
        return response()->json(['success' => true, 'data' => $registro]);
    }

    public function update(Request $request, $id)
    {
        $registro = NombreModelo::findOrFail($id);
        $request->validate([
            'campo1' => 'required|string|max:255',
        ]);
        $registro->update($request->validated());

        return response()->json([
            'success' => true,
            'data'    => $registro,
            'message' => 'Registro actualizado correctamente.',
        ]);
    }

    public function destroy($id)
    {
        NombreModelo::findOrFail($id)->delete();
        return response()->json(['success' => true, 'message' => 'Registro eliminado.']);
    }
}
```

### Rutas (routes/api.php)
```php
Route::prefix('nombre-modulo')->group(function () {
    Route::get('/',          [NombreController::class, 'index']);
    Route::post('/',         [NombreController::class, 'store']);
    Route::get('/{id}',      [NombreController::class, 'show']);
    Route::put('/{id}',      [NombreController::class, 'update']);
    Route::delete('/{id}',   [NombreController::class, 'destroy']);
});
```

---

## Frontend — Vue 2 + Vuexy

### Estructura de Archivos
```
src/
├── views/
│   └── modulo/
│       ├── Index.vue          ← lista principal con vs-table
│       ├── Form.vue           ← formulario en vs-popup o página
│       └── Show.vue           ← detalle (opcional)
├── api/
│   └── nombreModulo.js        ← todas las llamadas axios aquí
└── store/
    └── modules/
        └── nombreModulo.js    ← Vuex si hay estado global
```

### API Service (src/api/nombreModulo.js)
```js
import axios from '@/libs/axios'

const BASE = '/api/nombre-modulo'

export default {
  getAll:   (params) => axios.get(BASE, { params }),
  getOne:   (id)     => axios.get(`${BASE}/${id}`),
  create:   (data)   => axios.post(BASE, data),
  update:   (id, data) => axios.put(`${BASE}/${id}`, data),
  remove:   (id)     => axios.delete(`${BASE}/${id}`),
}
```

---

## Componentes Vuesax — Patrones Estándar

Ver `references/components.md` para ejemplos completos de cada componente.

### Resumen rápido de componentes disponibles:
- `vs-table` + `vs-th` + `vs-tr` + `vs-td` — tablas con search y sort integrado
- `vs-popup` — modales elegantes
- `vs-input` — inputs estilizados
- `vs-select` + `vs-option` — selects con búsqueda
- `vs-button` — botones con variantes (filled, border, flat, relief, gradient)
- `vs-chip` — badges/tags de estado
- `vs-card` — contenedores con sombra
- `vs-alert` — alertas inline
- `vs-tooltip` — tooltips
- `vs-pagination` — paginación
- `vs-dialog` — confirmaciones
- `vs-loading` — loading overlay

---

## Vista Index Estándar (CRUD completo)

```vue
<template>
  <div>
    <!-- Header -->
    <div class="vx-card p-6 mb-6">
      <div class="flex items-center justify-between mb-4">
        <h4 class="font-semibold text-lg">Gestión de [Módulo]</h4>
        <vs-button
          color="primary"
          type="filled"
          icon-pack="feather"
          icon="icon-plus"
          @click="openCreate"
        >
          Nuevo Registro
        </vs-button>
      </div>

      <!-- Filtros -->
      <div class="flex flex-wrap gap-4">
        <vs-input
          v-model="searchQuery"
          placeholder="Buscar..."
          icon-pack="feather"
          icon="icon-search"
          icon-no-border
          class="w-64"
        />
        <vs-select
          v-model="filtroEstado"
          placeholder="Estado"
          class="w-40"
        >
          <vs-option value="">Todos</vs-option>
          <vs-option value="1">Activo</vs-option>
          <vs-option value="0">Inactivo</vs-option>
        </vs-select>
      </div>
    </div>

    <!-- Tabla -->
    <div class="vx-card">
      <vs-table
        ref="table"
        :data="registros"
        :no-data-text="loading ? 'Cargando...' : 'No hay registros'"
        search
        pagination
        :max-items="perPage"
      >
        <template slot="header">
          <h4 class="font-semibold px-4 py-2">
            {{ total }} registros encontrados
          </h4>
        </template>

        <template slot="thead">
          <vs-th sort-key="campo1">Campo 1</vs-th>
          <vs-th sort-key="campo2">Campo 2</vs-th>
          <vs-th>Estado</vs-th>
          <vs-th>Acciones</vs-th>
        </template>

        <template slot-scope="{ data }">
          <vs-tr
            v-for="(row, i) in data"
            :key="i"
            :class="{ 'opacity-50': !row.activo }"
          >
            <vs-td :data="row.campo1">{{ row.campo1 }}</vs-td>
            <vs-td :data="row.campo2">{{ row.campo2 }}</vs-td>
            <vs-td>
              <vs-chip
                :color="row.activo ? 'success' : 'danger'"
                class="font-semibold"
              >
                {{ row.activo ? 'Activo' : 'Inactivo' }}
              </vs-chip>
            </vs-td>
            <vs-td>
              <div class="flex gap-2">
                <vs-button
                  size="small"
                  type="border"
                  color="warning"
                  icon-pack="feather"
                  icon="icon-edit"
                  @click="openEdit(row)"
                />
                <vs-button
                  size="small"
                  type="border"
                  color="danger"
                  icon-pack="feather"
                  icon="icon-trash-2"
                  @click="confirmDelete(row.id)"
                />
              </div>
            </vs-td>
          </vs-tr>
        </template>
      </vs-table>
    </div>

    <!-- Modal Crear / Editar -->
    <vs-popup
      :title="isEditing ? 'Editar Registro' : 'Nuevo Registro'"
      :active.sync="showPopup"
    >
      <div class="p-4">
        <div class="mb-4">
          <label class="vs-input--label">Campo 1 *</label>
          <vs-input
            v-model="form.campo1"
            placeholder="Ingrese valor..."
            class="w-full"
            :danger="!!errors.campo1"
            :danger-text="errors.campo1"
          />
        </div>

        <div class="flex justify-end gap-3 mt-6">
          <vs-button type="border" color="dark" @click="showPopup = false">
            Cancelar
          </vs-button>
          <vs-button
            color="primary"
            :disabled="saving"
            @click="save"
          >
            {{ saving ? 'Guardando...' : 'Guardar' }}
          </vs-button>
        </div>
      </div>
    </vs-popup>
  </div>
</template>

<script>
import NombreModuloApi from '@/api/nombreModulo'

export default {
  name: 'NombreModuloIndex',

  data() {
    return {
      registros:    [],
      total:        0,
      perPage:      15,
      searchQuery:  '',
      filtroEstado: '',
      loading:      false,
      saving:       false,
      showPopup:    false,
      isEditing:    false,
      form:         this.defaultForm(),
      errors:       {},
    }
  },

  created() {
    this.fetchData()
  },

  methods: {
    defaultForm() {
      return { id: null, campo1: '', campo2: '', activo: true }
    },

    async fetchData() {
      this.loading = true
      try {
        const { data } = await NombreModuloApi.getAll({
          search:  this.searchQuery,
          activo:  this.filtroEstado,
          per_page: this.perPage,
        })
        this.registros = data.data
        this.total     = data.total
      } catch (e) {
        this.$vs.notify({
          title: 'Error',
          text:  'No se pudo cargar la información.',
          color: 'danger',
          icon:  'icon-alert-circle',
          iconPack: 'feather',
        })
      } finally {
        this.loading = false
      }
    },

    openCreate() {
      this.form      = this.defaultForm()
      this.errors    = {}
      this.isEditing = false
      this.showPopup = true
    },

    openEdit(row) {
      this.form      = { ...row }
      this.errors    = {}
      this.isEditing = true
      this.showPopup = true
    },

    async save() {
      this.saving = true
      this.errors = {}
      try {
        if (this.isEditing) {
          await NombreModuloApi.update(this.form.id, this.form)
          this.notify('success', 'Registro actualizado correctamente.')
        } else {
          await NombreModuloApi.create(this.form)
          this.notify('success', 'Registro creado correctamente.')
        }
        this.showPopup = false
        this.fetchData()
      } catch (e) {
        if (e.response?.status === 422) {
          this.errors = e.response.data.errors || {}
        }
        this.notify('danger', 'Ocurrió un error al guardar.')
      } finally {
        this.saving = false
      }
    },

    confirmDelete(id) {
      this.$vs.dialog({
        type:    'confirm',
        color:   'danger',
        title:   'Confirmar eliminación',
        text:    '¿Estás seguro de eliminar este registro? Esta acción no se puede deshacer.',
        accept:  () => this.deleteRecord(id),
        acceptText: 'Eliminar',
        cancelText: 'Cancelar',
      })
    },

    async deleteRecord(id) {
      try {
        await NombreModuloApi.remove(id)
        this.notify('success', 'Registro eliminado.')
        this.fetchData()
      } catch {
        this.notify('danger', 'No se pudo eliminar el registro.')
      }
    },

    notify(color, text) {
      this.$vs.notify({
        title: color === 'success' ? 'Éxito' : 'Error',
        text,
        color,
        iconPack: 'feather',
        icon: color === 'success' ? 'icon-check-circle' : 'icon-alert-circle',
      })
    },
  },
}
</script>
```

---

## Convenciones de Estilo

### Colores Vuexy estándar
| Uso | Color |
|---|---|
| Acciones principales | `primary` |
| Éxito / Activo | `success` |
| Advertencia / Editar | `warning` |
| Error / Eliminar | `danger` |
| Secundario | `dark` |

### Botones por contexto
```html
<!-- Crear -->
<vs-button color="primary" type="filled" icon-pack="feather" icon="icon-plus">Nuevo</vs-button>

<!-- Editar -->
<vs-button color="warning" type="border" icon-pack="feather" icon="icon-edit" />

<!-- Eliminar -->
<vs-button color="danger" type="border" icon-pack="feather" icon="icon-trash-2" />

<!-- Cancelar -->
<vs-button color="dark" type="border">Cancelar</vs-button>

<!-- Ver detalle -->
<vs-button color="primary" type="flat" icon-pack="feather" icon="icon-eye" />
```

### Estados con vs-chip
```html
<vs-chip color="success">Activo</vs-chip>
<vs-chip color="danger">Inactivo</vs-chip>
<vs-chip color="warning">Pendiente</vs-chip>
<vs-chip color="primary">En Proceso</vs-chip>
```

---

## Referencias Adicionales

- `references/components.md` — Ejemplos completos de cada componente Vuesax
- `references/patterns.md` — Patrones avanzados: filtros, exportar Excel, carga de archivos
