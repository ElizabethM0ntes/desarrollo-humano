<template>
  <div class="container p-5">
    <v-app>
      <v-container>
        <h1 class="display-2">Índice de Desarrollo Humano en México</h1>
        
        <!-- AÑADIR AÑOS -->
        <v-row class="mb-4">
          <v-col cols="6">
            <v-card>
              <v-card-title class="headline">Añadir Nuevo Año</v-card-title>
              <v-card-text>
                <v-form @submit.prevent="addNewYear">
                  <v-text-field v-model="newYear" label="Año" required></v-text-field>
                  <v-btn type="submit" color="primary">Añadir Año</v-btn>
                </v-form>
              </v-card-text>
            </v-card>
          </v-col>
          
          <v-col cols="6">
            <v-card>
              <v-card-title class="headline">Años Existentes</v-card-title>
              <v-card-text>
                <v-list>
                  <v-list-item v-for="year in anios" :key="year">
                    <v-list-item-content>
                      <v-row align="center">
                        <v-col cols="10">{{ year }}</v-col>
                        <v-col cols="1">
                          <v-icon @click="openEditDialog(year)">mdi-pencil</v-icon>
                        </v-col>
                        <v-col cols="1">
                          <v-icon @click="deleteYear(year)">mdi-delete</v-icon>
                        </v-col>
                      </v-row>
                    </v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card-text>
            </v-card>

          </v-col>
        </v-row>

        
        <!-- FILTROS  -->
        <v-row>
          <v-col cols="4">
            <v-card>
              <v-card-title class="headline">Filtro por Estado</v-card-title>
              <v-card-text>
                <v-select
                  v-model="selectedState"
                  :items="estados.map(estado => estado.nombre)"
                  label="Estado"
                  multiple
                  @change="filterDataByStates"
                ></v-select>
              </v-card-text>
            </v-card>
          </v-col>


          <v-col cols="4">
            <v-card>
              <v-card-title class="headline">Filtro de Orden</v-card-title>
              <v-card-text>
                <v-select
                  v-model="sort"
                  :items="['alfabetico', 'ascendente', 'descendente']"
                  label="Ordenar datos"
                  @change="filterDataBySort"
                ></v-select>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="4">
            <v-card>
              <v-card-title class="headline">Filtro por Años</v-card-title>
              <v-card-text>
                <v-checkbox
                  v-for="anio in anios"
                  :key="anio"
                  v-model="selectedYears"
                  :label="anio.toString()"
                  :value="anio"
                  class="year-checkbox"
                  @change="filterDataByYears"
                ></v-checkbox>
              </v-card-text>
            </v-card>
          </v-col>

        </v-row>
        

        <!-- tabla de IDH -->
        <v-row>
          <v-col cols="12">
            <v-simple-table>
              <template v-slot:default>
                <thead>
                  <tr>
                    <th>Entidad</th>
                    <th>Año</th>
                    <th>IDH</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="entidad in paginatedEntidades" :key="entidad.id">
                    <td>{{ entidad.nombre }}</td>
                    <td>{{ entidad.anio }}</td>
                    <td>{{ entidad.idh }}</td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </v-col>
        </v-row>

        <!-- dialog para editar año -->
        <v-dialog v-model="editDialog" max-width="500">
          <v-card>
            <v-card-title class="headline">Editar Año</v-card-title>
            <v-card-text>
              <v-form @submit.prevent="saveEditedYear">
                <v-text-field v-model="editedYear" label="Nuevo Año" required></v-text-field>
                <v-btn type="submit" color="primary">Guardar Cambios</v-btn>
                <v-btn @click="closeEditDialog">Cancelar</v-btn>
              </v-form>
            </v-card-text>
          </v-card>
        </v-dialog>

        <!-- paginado -->
        <v-row>
          <v-col cols="12" class="text-center">
            <v-pagination v-model="currentPage" :length="totalPages" @input="changePage" />
          </v-col>
        </v-row>

      </v-container>

    </v-app>
  </div>
</template>

<!-- SCRIPTS -->

<script>
  export default {
    data() {
      return {
        newYear: '',
        selectedYearForEdit: null,
        editedYear: null,
        editDialog: false,
        estados: [
          { id: 1, nombre: 'Aguascalientes' },
          { id: 2, nombre: 'Baja California' },
          { id: 3, nombre: 'Baja California Sur' },
          { id: 4, nombre: 'Campeche' },
          { id: 5, nombre: 'Coahuila de Zaragoza' },
          { id: 6, nombre: 'Colima' },
          { id: 7, nombre: 'Chiapas' },
          { id: 8, nombre: 'Chihuahua' },
          { id: 9, nombre: 'Ciudad de México' },
          { id: 10, nombre: 'Durango' },
          { id: 11, nombre: 'Guanajuato' },
          { id: 12, nombre: 'Guerrero' },
          { id: 13, nombre: 'Hidalgo' },
          { id: 14, nombre: 'Jalisco' },
          { id: 15, nombre: 'México' },
          { id: 16, nombre: 'Michoacán de Ocampo' },
          { id: 17, nombre: 'Morelos' },
          { id: 18, nombre: 'Nayarit' },
          { id: 19, nombre: 'Nuevo León' },
          { id: 20, nombre: 'Oaxaca' },
          { id: 21, nombre: 'Puebla' },
          { id: 22, nombre: 'Querétaro' },
          { id: 23, nombre: 'Quintana Roo' },
          { id: 24, nombre: 'San Luis Potosí' },
          { id: 25, nombre: 'Sinaloa' },
          { id: 26, nombre: 'Sonora' },
          { id: 27, nombre: 'Tabasco' },
          { id: 28, nombre: 'Tamaulipas' },
          { id: 29, nombre: 'Tlaxcala' },
          { id: 30, nombre: 'Veracruz de Ignacio de la Llave' },
          { id: 31, nombre: 'Yucatán' },
          { id: 32, nombre: 'Zacatecas' }
        ], //Lista de estados
        anios: [2019, 2020, 2021], // Años
        selectedYears: [], // Años seleccionados
        selectedState: [], // Estados seleccionados
        sort: '', // Forma de ordenar los datos
        currentPage: 1, // Página actual
        itemsPerPage: 15, // Cantidad de elementos por página
        entidades: [ 
            { id: 1, nombre: 'Aguascalientes', idh: 0.823, anio: 2020 },
            { id: 2, nombre: 'Baja California', idh: 0.815, anio: 2021 },
            { id: 3, nombre: 'Baja California Sur', idh: 0.805, anio: 2020 },
            { id: 4, nombre: 'Campeche', idh: 0.779, anio: 2021 },
            { id: 5, nombre: 'Coahuila de Zaragoza', idh: 0.810, anio: 2020 },
            { id: 6, nombre: 'Colima', idh: 0.819, anio: 2021 },
            { id: 7, nombre: 'Chiapas', idh: 0.739, anio: 2020 },
            { id: 8, nombre: 'Chihuahua', idh: 0.826, anio: 2021 },
            { id: 9, nombre: 'Ciudad de México', idh: 0.916, anio: 2020 },
            { id: 10, nombre: 'Durango', idh: 0.789, anio: 2021 },
            { id: 11, nombre: 'Guanajuato', idh: 0.800, anio: 2020 },
            { id: 12, nombre: 'Guerrero', idh: 0.759, anio: 2021 },
            { id: 13, nombre: 'Hidalgo', idh: 0.785, anio: 2020 },
            { id: 14, nombre: 'Jalisco', idh: 0.821, anio: 2021 },
            { id: 15, nombre: 'México', idh: 0.805, anio: 2020 },
            { id: 16, nombre: 'Michoacán de Ocampo', idh: 0.771, anio: 2021 },
            { id: 17, nombre: 'Morelos', idh: 0.820, anio: 2020 },
            { id: 18, nombre: 'Nayarit', idh: 0.776, anio: 2021 },
            { id: 19, nombre: 'Nuevo León', idh: 0.849, anio: 2020 },
            { id: 20, nombre: 'Oaxaca', idh: 0.737, anio: 2021 },
            { id: 21, nombre: 'Puebla', idh: 0.784, anio: 2020 },
            { id: 22, nombre: 'Querétaro', idh: 0.849, anio: 2021 },
            { id: 23, nombre: 'Quintana Roo', idh: 0.794, anio: 2020 },
            { id: 24, nombre: 'San Luis Potosí', idh: 0.791, anio: 2021 },
            { id: 25, nombre: 'Sinaloa', idh: 0.813, anio: 2020 },
            { id: 26, nombre: 'Sonora', idh: 0.825, anio: 2021 },
            { id: 27, nombre: 'Tabasco', idh: 0.775, anio: 2020 },
            { id: 28, nombre: 'Tamaulipas', idh: 0.826, anio: 2021 },
            { id: 29, nombre: 'Tlaxcala', idh: 0.804, anio: 2020 },
            { id: 30, nombre: 'Veracruz de Ignacio de la Llave', idh: 0.759, anio: 2021 },
            { id: 31, nombre: 'Yucatán', idh: 0.830, anio: 2020 },
            { id: 32, nombre: 'Zacatecas', idh: 0.746, anio: 2019 }

          ], // Lista de entidades con datos
      };
  },
    
    computed: {
      // Entidades filtradas y ordenadas
      filteredEntidades() {
        let filtered = this.entidades.slice();

        if (this.selectedYears.length > 0) {
          filtered = filtered.filter(entidad => this.selectedYears.includes(entidad.anio));
        }

        if (this.selectedState.length > 0) {
          filtered = filtered.filter(entidad => this.selectedState.includes(entidad.nombre));
        }

        if (this.sort === 'alfabetico') {
          filtered = filtered.sort((a, b) => a.nombre.localeCompare(b.nombre));
        } else if (this.sort === 'ascendente') {
          filtered = filtered.sort((a, b) => a.idh - b.idh);
        } else if (this.sort === 'descendente') {
          filtered = filtered.sort((a, b) => b.idh - a.idh);
        }

        return filtered;
      },
      // Total de páginas para la paginación
      totalPages() {
        return Math.ceil(this.filteredEntidades.length / this.itemsPerPage);
      },
      // Datos paginados según la página actual
      paginatedEntidades() {
        const startIndex = (this.currentPage - 1) * this.itemsPerPage;
        const endIndex = startIndex + this.itemsPerPage;
        return this.filteredEntidades.slice(startIndex, endIndex);
      },
    },
    
  methods: {
    // Filtra y ordena los datos de las entidades según la opción de orden seleccionada.
      filterDataBySort() {
        if (this.sort === 'alfabetico') {
          this.filteredEntidades = this.filteredEntidades.sort((a, b) => a.nombre.localeCompare(b.nombre));
        } else if (this.sort === 'ascendente') {
          this.filteredEntidades = this.filteredEntidades.sort((a, b) => a.idh - b.idh);
        } else if (this.sort === 'descendente') {
          this.filteredEntidades = this.filteredEntidades.sort((a, b) => b.idh - a.idh);
        }
      },

      filterDataByStates() {
        if (this.selectedState.length === 0) {
          // Si no se ha seleccionado ningún estado, mostrar todos los datos
          this.filteredEntidades = this.entidades;
        } else {
          // Filtrar los datos por estados seleccionados
          this.filteredEntidades = this.entidades.filter(entidad =>
            this.selectedState.includes(entidad.nombre)
          );
        }
        // Reiniciar la página al cambiar el filtro
        this.currentPage = 1;
    },
      
      // agregar año
        addNewYear() {
          if (this.newYear && !this.anios.includes(this.newYear)) {
            this.anios.push(this.newYear);
            this.newYear = '';
          }
    },
      // borrar año

        deleteYear(year) {
          const index = this.anios.indexOf(year);
          if (index !== -1) {
            this.anios.splice(index, 1);
            if (this.selectedYears.includes(year)) {
              this.selectedYears = this.selectedYears.filter(selectedYear => selectedYear !== year);
            }
          }
        },
        

        // editar año
        openEditDialog(year) {
          this.selectedYearForEdit = year;
          this.editedYear = year;
          this.editDialog = true;
        },
        
        closeEditDialog() {
          this.editDialog = false;
          this.selectedYearForEdit = '';
          this.editedYear = '';
        },
        
        saveEditedYear() {
          if (this.editedYear && !this.anios.includes(this.editedYear)) {
            const index = this.anios.indexOf(this.selectedYearForEdit);
            if (index !== -1) {
              this.anios[index] = this.editedYear;
            }
            this.closeEditDialog();
          }
        },

        // Cambiar la página actual
        changePage(newPage) {
          this.currentPage = newPage;
        },

        // Generar valores aleatorios para el IDH
        generateRandomData() {
          for (let i = 0; i < this.anios.length; i++) {
            for (const estado of this.estados) {
              const randomIdh = Math.random() * (1 - 0) + 0;
              const formattedIdh = Number(randomIdh.toFixed(2));
              this.entidades.push({
                id: this.entidades.length + 1,
                nombre: estado.nombre,
                idh: formattedIdh,
                anio: this.anios[i],
              });
            }
          }
    },
        
        filterDataByYears() {
          this.currentPage = 1; // Reiniciar la página al cambiar la selección
        },

    },
    
    created() {
      this.generateRandomData();
    },
  };
</script>