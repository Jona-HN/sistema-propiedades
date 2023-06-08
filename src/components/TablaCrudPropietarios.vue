<template>
  <v-data-table
    :headers="headers"
    :items="propietarios"
    :sort-by="[{ key: 'id', order: 'asc' }]"
    class="elevation-1"
    fixed-header>
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Propietarios</v-toolbar-title>
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
              Agregar propietario
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
                      v-model="editedPropietario.personaId"
                      label="Id persona"
                    ></v-text-field>
                </v-row>
                <v-row>
                  <v-text-field
                      v-model="editedPropietario.propiedadId"
                      label="Id propiedad"
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
            <v-card-title class="text-h6">¿Está seguro de eliminar este propietario?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancelar</v-btn>
              <v-btn color="blue-darken-1" variant="text" @click="deletePropietarioConfirm">Confirmar</v-btn>
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
        @click="editPropietario(item.raw)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        size="small"
        @click="deletePropietario(item.raw)"
      >
        mdi-delete
      </v-icon>
    </template>

    <template v-slot:no-data>
      No hay propietarios registrados
    </template>
  </v-data-table>
</template>

<script setup>
  import { ref, onMounted, computed, watch, nextTick } from 'vue';

  const propietarios = ref([]);
  const headers = ref([
    {
      title: 'Id',
      key: 'id',
    },
    {
      title: 'Id persona',
      key: 'personaId'
    },
    {
      title: 'Id propiedad',
      key: 'propiedadId'
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
  const editedPropietario = ref({
    id: 0,
    personaId: '',
    propietarioId: ''
  });
  const defaultPropietario = {
    id: 0,
    personaId: '',
    propietarioId: ''
  };

  const formTitle = computed({
    get() {
      return editedIndex.value === -1 ? 'Agregar propietario' : 'Editar propietario';
    }
  })

  const close = async () => {
    dialog.value = false;
    await nextTick(() => {
      editedPropietario.value = Object.assign({}, defaultPropietario.value);
      editedIndex.value = -1;
    })
  }

  const save = () => {
    if (editedIndex.value > -1) {
      fetch('https://localhost:2800/propietarios/update', {
        method: 'PATCH',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(editedPropietario.value)
      })
        .then(response => response.json())
        .then(json => {
          propietarios.value = json;
          console.log(json);
        });
    }
    else {
      fetch('https://localhost:2800/propietarios/add', {
        method: 'POST',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(editedPropietario.value)
      })
        .then(response => response.json())
        .then(json => {
          propietarios.value = json;
          console.log(json);
        });
    }

    close();
  }

  const closeDelete = async () => {
    dialogDelete.value = false;
    await nextTick(() => {
      editedPropietario.value = Object.assign({}, defaultPropietario.value);
      editedIndex.value = -1;
    })
  }

  const deletePropietarioConfirm = () => {
    fetch('https://localhost:2800/propietarios/delete/' + editedPropietario.value.id, {
        method: 'DELETE',
      })
        .then(response => response.json())
        .then(json => {
          propietarios.value = json;
          console.log(json);
        });

    closeDelete();
  }

  const editPropietario = (propietario) => {
    editedIndex.value = propietarios.value.indexOf(propietario);
    editedPropietario.value = Object.assign({}, propietario);
    dialog.value = true;
  }

  const deletePropietario = (propietario) => {
    editedIndex.value = propietarios.value.indexOf(propietario);
    editedPropietario.value = Object.assign({}, propietario);
    dialogDelete.value = true;
  }

  onMounted(initialize);

  function initialize() {
    fetch('https://localhost:2800/propietarios')
      .then(response => response.json())
      .then(json => {
        propietarios.value = json;
      })
      .catch(err => {
        console.log(err);
      });
  }
</script>