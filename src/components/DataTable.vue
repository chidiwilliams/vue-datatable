<template>
  <div class="main">
    <h2>#DATA-TABLE</h2>

    <label for="file" class="file-button">
        Upload CSV
    </label>
    <input type="file" id="file" 
           @change="handleData" name="data" accept=".csv">

    <div class="holder">
      <table>
      <thead>
        <tr>
          <th>
            S/N
          </th>
          <th v-for="header in headers" :key="header.id"> 
            {{ header.text }} 
            <span class="sortUp" role="button" @click="sortUp(header)">🔼</span>
            <span class="sortDown" role="button" @click="sortDown(header)">🔽</span>
          </th>
        </tr>
      </thead>

      <tbody>
        <tr>
          <td></td>
          <td v-for="header in headers" :key="header.id" class="filter">
            <input type="search" @keyup="filter(header, $event)">
          </td>
        </tr>
        <tr v-for="(row, index) of rowsCopy" :key="row.id">
          <td> {{index + 1}} </td>
          <td v-for="header in headers" :key="header.id" >
            {{ row[header.text] }}
          </td>
        </tr>
      </tbody>
    </table>
    </div>

  </div>
</template>

<script>

export default {
  name: 'DataTable',
  data: function () {
    return {
      headers: [{text:'fake header1'},
                {text:'fake header2'},
                {text:'fake header3'}],

      rows: [{"fake header1": "whatever", "fake header2": "whatever",
             "fake header3": "whatever"},
             {"fake header1": "whatever", "fake header2": "whatever",
             "fake header3": "whatever"},
             {"fake header1": "whatever", "fake header2": "whatever",
             "fake header3": "whatever"}],

      rowsCopy: []
    }
  },
  methods: {
    handleData: async function(event) {
      // Taking advantage of await/async to make asyc code behave
      const d = await this.convertToJSON(event.target.files[0]);
      this.rows = d["rows"];
      this.rowsCopy = d["rows"];
      this.headers = d["headers"]
    },
    convertToJSON: function(file) {
      return new Promise( function(resolve) {
        let reader = new FileReader();

        reader.readAsText(file);

        reader.onload = function(event) {
          const csvArray = event.target.result.split("\n")
          const headers = csvArray[0].split(/,(?=\w|\d)/) //split using `comma` as delimiter
          const rows = csvArray.slice(1,-1);

          // build an object and get around `this` hell
          this.createObject = function(head, arr) {
            let obj = {};

            for (let i=0; i<head.length; i++) {
              obj[head[i]] = arr[i]
            }
            return obj
          };

          this.buildCSVArray = function(str) {
            let arr = [];
            let quoteCount = 0;
            let doubleQuoteCount = 0;
            let charCount = 0;

            for (let i=0; i<str.length; i++) {
              if (str[i] == `"`) { doubleQuoteCount++ }
              if (str[i] == `'`) { quoteCount++ }
              if ( (str[i] == `,` || i == str.length -1) && quoteCount%2 == 0 && doubleQuoteCount%2 == 0 ) {
                let ar = str.slice(i-charCount,i);
                arr.push(ar);
                charCount = -1;
              }
              charCount++
            }
            return arr
          }

          for (let i=0; i<rows.length; i++) {
            rows[i] = this.createObject(headers, this.buildCSVArray(rows[i]))
          }

          resolve({headers: headers.map((el) => { return {text:el}}), rows: rows});
        }
        // end of reader.onload
      })
    },
    sortUp: function(header) {
      // get down to the column
      let headerKey = header.text;
      this.rowsCopy.sort( function (el1, el2) { 
        if (parseFloat(el1[headerKey]) && parseFloat(el2[headerKey]) && el1[headerKey].indexOf('-') == -1) {
          // if a number is encountered
          return parseFloat(el1[headerKey]) - parseFloat(el2[headerKey]);
        } else {
          if (el1[headerKey] < el2[headerKey])
            return -1;
          if (el1[headerKey] > el2[headerKey])
            return 1;
        }
        return 0
      });
    },
    sortDown: function(header) {
      // get down to the column
      let headerKey = header.text;
      this.rowsCopy.sort( function (el1, el2) {
        if (parseFloat(el1[headerKey]) && parseFloat(el2[headerKey]) && el1[headerKey].indexOf('-') == -1) {
          // if a number is encountered
          return parseFloat(el2[headerKey]) - parseFloat(el1[headerKey]);
        } else {
          if (el1[headerKey] > el2[headerKey])
            return -1;
          if (el1[headerKey] < el2[headerKey])
            return 1;
         }
         return 0
      });
    },
    filter: function(header, event) {
      let keyWord = header.text;
      let searchVal = event.target.value;
      this.resolveFilter(keyWord, searchVal, this.rows);
    },
    resolveFilter: async function(keyWord, searchVal, rows) {
      this.rowsCopy = await this.feedResolveFilter(keyWord, searchVal, rows);
    },
    feedResolveFilter: function(keyWord, searchVal, rows) {
      return new Promise(function(resolve) {
        searchVal = new RegExp(searchVal, "i");
        resolve(rows.filter(function (el) { return el[keyWord].search(searchVal) !== -1 }))
      })
    }
  }
}
</script>

<style scoped>
.main {
  background-color: rgb(253, 252, 252);
  margin: 1em;
  padding: 3em;
  border: 1px solid rgb(143, 141, 141);
  border-radius: 0.8em;
}

input[type=file]{
  display: none;
}


.file-button {
  font-family: 'Karla', sans-serif;
  color:white;
  padding: 0.6em 1em;
  background-color: #287AE6;
  cursor: pointer;
  display: inline-block;
  margin-bottom: 1em;
  border-radius: 0.4em;
  box-shadow: 0 4px 8px 1px rgba(0,0,0,.14);
}

.file-button:hover {
  background-color: rgb(29, 89, 167);
}

.holder {
  /* width: 500px; */
  overflow-x: auto;
}

table {
  border-collapse: collapse;
  box-shadow: 0 5px 5px -3px rgba(0,0,0,.2), 0 8px 10px 1px rgba(0,0,0,.14), 0 3px 14px 2px rgba(0,0,0,.12);
  margin-top: 20px;
}

table, th, td {
  border: 1px solid #414141;
  padding: 0.5em;
}

th, td {
  padding: 10px 7px;
  font-weight: 100 !important;
  /* white-space: nowrap; */
}

thead {
  background-color: #287AE6;
  color: white;
}

th > span {
  text-align: center;
  width: 11px;
  border-radius: 11px;
  padding-left: 4px;
  padding-right:4px
}

th > span:hover {
  cursor: pointer;
  background-color: rgb(29, 89, 167);
}

td > input {
  padding-bottom: 10px;
  margin: 0 auto;
  display: inline-block
}

.filter {
  text-align: center
}
</style>
