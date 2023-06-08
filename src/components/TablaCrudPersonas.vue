<template>
  <v-data-table
    :headers="headers"
    :items="personas"
    :sort-by="[{ key: 'id', order: 'asc' }]"
    class="elevation-1"
    fixed-header>
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Personas</v-toolbar-title>
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
              Agregar persona
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
                      v-model="editedPersona.nombre"
                      label="Nombre"
                    ></v-text-field>
                </v-row>
                <v-row>
                  <v-text-field
                      v-model="editedPersona.rfc"
                      label="RFC"
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
            <v-card-title class="text-h6">¿Está seguro de eliminar a esta persona?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancelar</v-btn>
              <v-btn color="blue-darken-1" variant="text" @click="deletePersonaConfirm">Confirmar</v-btn>
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
        @click="editPersona(item.raw)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        size="small"
        @click="deletePersona(item.raw)"
      >
        mdi-delete
      </v-icon>
    </template>

    <template v-slot:no-data>
      No hay personas registradas
    </template>
  </v-data-table>
</template>

<script setup>
  import { ref, onMounted, computed, watch, nextTick } from 'vue';

  const personas = ref([]);
  const headers = ref([
    {
      title: 'Id',
      key: 'id',
    },
    {
      title: 'Nombre',
      key: 'nombre'
    },
    {
      title: 'RFC',
      key: 'rfc',
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
  const editedPersona = ref({
    id: 0,
    nombre: '',
    rfc: '',
  });
  const defaultPersona = {
    id: 0,
    nombre: '',
    rfc: '',
  };

  const formTitle = computed({
    get() {
      return editedIndex.value === -1 ? 'Agregar persona' : 'Editar persona';
    }
  })

  const close = async () => {
    dialog.value = false;
    await nextTick(() => {
      editedPersona.value = Object.assign({}, defaultPersona.value);
      editedIndex.value = -1;
    })
  }

  const save = () => {
    if (editedIndex.value > -1) {
      fetch('https://localhost:2800/personas/update', {
        method: 'PATCH',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(editedPersona.value)
      })
        .then(response => response.json())
        .then(json => {
          personas.value = json;
          console.log(json);
        });
    }
    else {
      fetch('https://localhost:2800/personas/add', {
        method: 'POST',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(editedPersona.value)
      })
        .then(response => response.json())
        .then(json => {
          personas.value = json;
          console.log(json);
        });
    }

    close();
  }

  const closeDelete = async () => {
    dialogDelete.value = false;
    await nextTick(() => {
      editedPersona.value = Object.assign({}, defaultPersona.value);
      editedIndex.value = -1;
    })
  }

  const deletePersonaConfirm = () => {
    fetch('https://localhost:2800/personas/delete/' + editedPersona.value.id, {
        method: 'DELETE',
      })
        .then(response => response.json())
        .then(json => {
          personas.value = json;
          console.log(json);
        });

    closeDelete();
  }

  const editPersona = (persona) => {
    editedIndex.value = personas.value.indexOf(persona);
    editedPersona.value = Object.assign({}, persona);
    dialog.value = true;
  }

  const deletePersona = (persona) => {
    editedIndex.value = personas.value.indexOf(persona);
    editedPersona.value = Object.assign({}, persona);
    dialogDelete.value = true;
  }

  onMounted(initialize);

  function initialize() {
    fetch('https://localhost:2800/personas')
      .then(response => response.json())
      .then(json => {
        personas.value = json;
      })
      .catch(err => {
        console.log(err);
      });
  }
</script>