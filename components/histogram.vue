<template>
     <v-sheet 
  class="d-flex align-content-end justify-center flex-wrap  "
   style="z-index: 1; background: transparent; " 

  >
  <div style="align-items: center; min-width: 100%; margin-left: 150px; margin-top: 40px; margin-bottom: 25px;">
    <v-file-input
    show-size
    counter
    center-affix="true"
    multiple
    label="Cargue su archivo"
    ref="fileInput"
    @change="handleFileChange"
    @clear="clearFileInput"
    style="min-width: 300px; max-width: 500px;  color: #000000;"
  ></v-file-input>
</div>
      <v-card 
      v-for="(stat, index) in columnStats" :key="index"
      class="ma-4 pa-4"
      style="background-color: rgb(131, 202, 219);margin: 10px; gap: 50%;"
      
      >
        <v-card-title>{{ columnTitles[index] }}</v-card-title>
        <v-card-text>
          
          <p>Mínimo: {{ stat.min }}</p>
          <p>Máximo: {{ stat.max }}</p>
          <p>Media: {{ stat.mean }}</p>
          <p>Mediana: {{ stat.median }}</p>
          <p>Moda: {{ stat.mode }}</p>
          <p>Desviación estándar: {{ stat.standardDeviation }}</p>
          <p>Tamaño de la población: {{ columnData[0].length - 1 }}</p>
          
        </v-card-text>

        <v-dialog
        transition="dialog-bottom-transition"
        width="80%"
        
      >
        <template v-slot:activator="{ props }">
          <v-btn
            color="blue-lighten-5"
            v-bind="props"
           
          >Mostrar Histograma</v-btn>
        </template>
    
        <template v-slot:default="{ isActive }">
         
          <v-card>
            <v-toolbar
              color="blue-lighten-3"
              
            ></v-toolbar>
            <v-card-text>

              <center>
              <Bar
          id="my-chart-id"
          :options="chartOptions"
          :data="getChartData(index)"
        />
              </center>
           
            </v-card-text>
           
            <v-card-actions class="justify-end">
        
              <v-btn
                variant="text"
                @click="isActive.value = false"
              >Close</v-btn>
            </v-card-actions>
          </v-card>
         
        </template>
      
      </v-dialog>

        
       
      </v-card>
    
  </v-sheet>
  </template>
  <style>
  .p{
    font-weight: bold;
    }
  </style>
  <script >
  import * as XLSX from 'xlsx';
  import * as ss from 'simple-statistics';
  import { Bar } from 'vue-chartjs'
  import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
    
  ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)
  
  
  export default {
    name: 'BarChart',
    components: { Bar },

    data() {
      return {
        dialog: false,
        chartData: {
            labels: [],
            datasets: [],
          },
          chartOptions: {
            responsive: true
          },
        selectedFile: null,
        columnData: [],
        columnStats: [],
        columnTitles: [],
      
      };
    },
    methods: {
      
    handleFileChange(event) {
      this.selectedFile = event.target.files ? event.target.files[0] : null;
  
  
    this.columnData = [];
    this.columnStats = [];
    this.columnTitles = [];
  
      let reader = new FileReader();
      reader.onload = (e) => {
        let data = new Uint8Array(e.target.result);
        let workbook = XLSX.read(data, {type: 'array'});
        this.readColumns(workbook);
      };
      reader.readAsArrayBuffer(this.selectedFile);
    },
    getChartData(index) {
        const data = this.columnData[index];
        const labels = [...new Set(data)].sort((a,b)=>a-b); // Obtén los datos únicos para las etiquetas
        const datasetsData = labels.map(label => data.filter(item => item === label).length); // Cuenta las ocurrencias de cada etiqueta
  
        return {
          labels: labels,
          datasets: [{ data: datasetsData, label: this.columnTitles[index], backgroundColor: '#90CAF9',hoverBorderWidth: 2, borderColor: '#0D47A1',pointStyle:'star' }]
        };
      },
    readColumns(workbook) {
      let firstSheetName = workbook.SheetNames[0];
      let worksheet = workbook.Sheets[firstSheetName];
      
      // Convertir la hoja de trabajo en un array de arrays
      let data = XLSX.utils.sheet_to_json(worksheet, {header: 1});
      
      // Crear un array para cada columna
      let columns = [];
      for (let i = 0; i < data[0].length; i++) {
        let columnData = data.map(row => row[i]);
        this.columnTitles.push(columnData[0]); // Guardamos el titulo
        columns.push(columnData.slice(1));
      }
      
      // Guardar los datos de las columnas en una variable
      this.columnData = columns;
      console.log('Las coumnas son:',this.columnData)
  
      // Calcular estadísticas descriptivas para cada columna
  
      this.columnStats = this.columnData.map(column => {
  
        const numericValues = column.filter(value => !isNaN(value));
  
        return {
          min: ss.min(numericValues),
          max: ss.max(numericValues),
          mean: ss.mean(numericValues),
          median: ss.median(numericValues),
          mode: ss.mode(numericValues),
          standardDeviation: ss.standardDeviation(numericValues).toFixed(4)
        };
      });
  
      console.log('Las estadisticas de las columnas son:',this.columnStats)
  
    },
  }
  ,
  };
  </script>