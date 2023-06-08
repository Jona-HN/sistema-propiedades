<template>
  <v-data-table
    :headers="headers"
    :items="propiedades"
    :sort-by="[{ key: 'id', order: 'asc' }]"
    class="elevation-1"
    fixed-header>
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Propiedades</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ props }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="props"
            >
              Agregar propiedad
            </v-btn>
          </template>
          
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-text-field
                      v-model="editedPropiedad.cve_catastral"
                      label="Clave catastral"
                    ></v-text-field>
                </v-row>
                <v-row>
                  <v-text-field
                      v-model="editedPropiedad.descripcion"
                      label="Descripción"
                    ></v-text-field>
                </v-row>
                <v-row>
                  <v-text-field
                      v-model="editedPropiedad.direccion"
                      label="Dirección"
                    ></v-text-field>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="close"
              >
                Cancelar
              </v-btn>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="save"
              >
                Aceptar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h6">¿Está seguro de eliminar esta propiedad?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancelar</v-btn>
              <v-btn color="blue-darken-1" variant="text" @click="deletePropiedadConfirm">Confirmar</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    
    <template v-slot:item.actions="{ item }">
      <v-icon
        size="small"
        class="me-2"
        @click="editPropiedad(item.raw)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        size="small"
        @click="deletePropiedad(item.raw)"
      >
        mdi-delete
      </v-icon>
    </template>

    <template v-slot:no-data>
      No hay propiedades registradas
    </template>
  </v-data-table>
</template>

<script setup>
  import { ref, onMounted, computed, watch, nextTick } from 'vue';

  const propiedades = ref([]);
  const headers = ref([
    {
      title: 'Id',
      key: 'id',
    },
    {
      title: 'Clave catastral',
      key: 'cve_catastral'
    },
    {
      title: 'Descripción',
      key: 'descripcion',
    },
    {
      title: 'Dirección',
      key: 'direccion',
    },
    {
      title: 'Creado el',
      key: 'createdAt',
    },
    {
      title: 'Actualizado el',
      key: 'updatedAt',
    },
    {
      title: 'Acciones',
      key: 'actions',
      sortable: false
    },
  ]);
  
  const dialog = ref(false);
  const dialogDelete = ref(false);

  watch(dialog, (val) => {
    val || close();
  });

  watch(dialogDelete, (val) => {
    val || closeDelete();
  });

  const editedIndex = ref(-1);
  const editedPropiedad = ref({
    id: 0,
    cve_catastral: '',
    descripcion: '',
    direccion: ''
  });
  const defaultPropiedad = {
    id: 0,
    cve_catastral: '',
    descripcion: '',
    direccion: ''
  };

  const formTitle = computed({
    get() {
      return editedIndex.value === -1 ? 'Agregar propiedad' : 'Editar propiedad';
    }
  })

  const close = async () => {
    dialog.value = false;
    await nextTick(() => {
      editedPropiedad.value = Object.assign({}, defaultPropiedad.value);
      editedIndex.value = -1;
    })
  }

  const save = () => {
    if (editedIndex.value > -1) {
      fetch('https://localhost:2800/propiedades/update', {
        method: 'PATCH',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(editedPropiedad.value)
      })
        .then(response => response.json())
        .then(json => {
          propiedades.value = json;
          console.log(json);
        });
    }
    else {
      fetch('https://localhost:2800/propiedades/add', {
        method: 'POST',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(editedPropiedad.value)
      })
        .then(response => response.json())
        .then(json => {
          propiedades.value = json;
          console.log(json);
        });
    }

    close();
  }

  const closeDelete = async () => {
    dialogDelete.value = false;
    await nextTick(() => {
      editedPropiedad.value = Object.assign({}, defaultPropiedad.value);
      editedIndex.value = -1;
    })
  }

  const deletePropiedadConfirm = () => {
    fetch('https://localhost:2800/propiedades/delete/' + editedPropiedad.value.id, {
        method: 'DELETE',
      })
        .then(response => response.json())
        .then(json => {
          propiedades.value = json;
          console.log(json);
        });

    closeDelete();
  }

  const editPropiedad = (persona) => {
    editedIndex.value = propiedades.value.indexOf(persona);
    editedPropiedad.value = Object.assign({}, persona);
    dialog.value = true;
  }

  const deletePropiedad = (persona) => {
    editedIndex.value = propiedades.value.indexOf(persona);
    editedPropiedad.value = Object.assign({}, persona);
    dialogDelete.value = true;
  }

  onMounted(initialize);

  function initialize() {
    fetch('https://localhost:2800/propiedades')
      .then(response => response.json())
      .then(json => {
        propiedades.value = json;
      })
      .catch(err => {
        console.log(err);
      });
  }
</script>