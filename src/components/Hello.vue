<template>
  <div style="text-align: center">
    <img class="no-print" src="../assets/KFCSoGood.png" style="width: 200px" /></br>
    <a class="no-print" href="https://goo.gl/forms/DkGHluETt2520Who2">FORMULARZ</a>
    <v-client-table :data="tableData" :columns="columns" :options="options"></v-client-table>
  </div>
</template>

<script>
import moment from "moment"
import linq from "mini-linq-js"

export default {
    name: 'hello',
    data () {
      return {
        columns: ['imie','nazwisko', 'data', 'wt', 'śr', 'czw', 'pt', 'sob', 'nd', 'pon'],
        tableData: [],
        options: {
        // see the options API
        },
        apiResponse: null,
        msg: 'Welcome to Your Vue.js App',
        apiUrl: "https://sheets.googleapis.com/v4/spreadsheets/1U88F18BsvBsRqDaVSzuN_zfeWt1vKEWLrmrxji-u_D4/values/A1:AG150?key=AIzaSyBA9lRMwPpaH1VdCUqef8ySzs62USv0lLQ"
      }
    },
    mounted: function(){
        this.updateApiResponse();
         
        setInterval(function(){
          this.updateApiResponse();
        }.bind(this), 10000);
    },
    methods: {
      updateApiResponse: function() {
        this.$http.get(this.apiUrl).then(
          response => 
          {
            this.apiResponse = response.body;
          }, 
          response => 
          {
            throw response;
          }
        ); 
      },
      updateTable: function(){
        var updatedTableData = [];

        let data = this.apiResponse.values;

        if(!data)
        {
          return;
        }

        if(data.length == 1)
        {
          this.tableData = updatedTableData;
        }
        
        const sygnaturaCzasowa = "Sygnatura czasowa";
        const imie = "Imię";
        const nazwisko = "Nazwisko";

        let titleRow = data[0];

        let imieColumnIndex = titleRow.indexOf(imie);
        let nazwiskoColumnIndex = titleRow.indexOf(nazwisko);
        let sygnaturaColumnIndex = titleRow.indexOf(sygnaturaCzasowa);

        for(let row = 1; row < data.length; row++)
        {
          let dataRow = data[row];

          if(!!dataRow[sygnaturaColumnIndex])
          {
            let pushRow = 
            { 
              imie: dataRow[imieColumnIndex],
              nazwisko: dataRow[nazwiskoColumnIndex],
              data: dataRow[sygnaturaColumnIndex]
            };

            pushRow.wt = this.getDataForDay(dataRow, titleRow, 1);
            pushRow["śr"] = this.getDataForDay(dataRow, titleRow, 2);
            pushRow.czw = this.getDataForDay(dataRow, titleRow, 3);
            pushRow.pt = this.getDataForDay(dataRow, titleRow, 4);
            pushRow.sob = this.getDataForDay(dataRow, titleRow, 5);
            pushRow.nd = this.getDataForDay(dataRow, titleRow, 6);
            pushRow.pon = this.getDataForDay(dataRow, titleRow, 7);

            updatedTableData.push(pushRow);
          }
        }
        
        this.tableData = updatedTableData;
      },
      getDataForDay: function(dataRow, titleRow, dayIndex){
        const fullOff = "FULL/OFF";
        const uwagi = "Uwagi";
        const godzinaOd = "Godzina od";
        const godzinaDo = "Godzina do";

        let fullOffIndex = titleRow.indexOf(dayIndex + " " + fullOff);
        let uwagiIndex = titleRow.indexOf(dayIndex + " " + uwagi);

        let choice = dataRow[fullOffIndex];
        let info = dataRow[uwagiIndex];

        if(choice == "FULL" || choice == "OFF")
        {
          return choice + (!!info ? " (" + info + ")" : "");
        }

        let godzinaOdIndex = titleRow.indexOf(dayIndex + " " + godzinaOd);
        let godzinaDoIndex = titleRow.indexOf(dayIndex + " " + godzinaDo);

        return dataRow[godzinaOdIndex] + " - " + dataRow[godzinaDoIndex]  + (!!info ? " (" + info + ")" : "");
      }
    },
    watch: {
      apiResponse: function(){
        this.updateTable();
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
    @media print
    {    
        @page { margin: 0; }
        body { margin: 1.6cm; }
        
        .no-print
        {
            display: none !important;
        }

        .VueTables__search
        {
          display: none;
        }

        * {
          -webkit-print-color-adjust: exact;
        }
    }

    table.VueTables__table {
    border: 1px solid #1C6EA4;
    background-color: #EEEEEE;
    width: 100%;
    text-align: left;
    border-collapse: collapse;
    }
    table.VueTables__table td, table.VueTables__table th {
    border: 1px solid #AAAAAA;
    padding: 3px 2px;
    }
    table.VueTables__table tbody td {
    font-size: 13px;
    }
    table.VueTables__table tr:nth-child(even) {
    background: #D0E4F5;
    }
    table.VueTables__table thead {
    background: #1C6EA4;
    background: -moz-linear-gradient(top, #5592bb 0%, #327cad 66%, #1C6EA4 100%);
    background: -webkit-linear-gradient(top, #5592bb 0%, #327cad 66%, #1C6EA4 100%);
    background: linear-gradient(to bottom, #5592bb 0%, #327cad 66%, #1C6EA4 100%);
    border-bottom: 2px solid #444444;
    }
    table.VueTables__table thead th {
    font-size: 15px;
    font-weight: bold;
    color: #FFFFFF;
    border-left: 2px solid #D0E4F5;
    }
    table.VueTables__table thead th:first-child {
    border-left: none;
    }

    table.VueTables__table tfoot {
    font-size: 14px;
    font-weight: bold;
    color: #FFFFFF;
    background: #D0E4F5;
    background: -moz-linear-gradient(top, #dcebf7 0%, #d4e6f6 66%, #D0E4F5 100%);
    background: -webkit-linear-gradient(top, #dcebf7 0%, #d4e6f6 66%, #D0E4F5 100%);
    background: linear-gradient(to bottom, #dcebf7 0%, #d4e6f6 66%, #D0E4F5 100%);
    border-top: 2px solid #444444;
    }
    table.VueTables__table tfoot td {
    font-size: 14px;
    }
    table.VueTables__table tfoot .links {
    text-align: right;
    }
    table.VueTables__table tfoot .links a{
    display: inline-block;
    background: #1C6EA4;
    color: #FFFFFF;
    padding: 2px 8px;
    border-radius: 5px;
    }
</style>
