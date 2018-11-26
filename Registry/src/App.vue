<template>
  <v-app id="app">
      <h1>{{kiiras}}</h1>
	  <label class="text-reader" v-if="!loading && view=='list'">
	  <i class="material-icons">backup</i>
	  <input type="file" @change="loadTextFromFile">
	  </label>
	  <label class="text-reader" v-if="loading && view=='list'">
		  <span slot="loader" class="custom-loader">
			 <v-icon light>cached</v-icon>
		  </span>
	  </label>
      </div>
      <div class="c1" v-if="view=='reszl'">
        <v-layout row v-for="(value, key) in record">
          <v-layout row>
            <v-flex text-xs-right xs6 reszl>
              <b>{{key}}:</b>
            </v-flex>
            <v-flex  text-xs-left xs6>
              &nbsp;
              <i>{{ value }}</i>
            </v-flex>
          </v-layout>
        </v-layout>
        <br>
        <hr>
        <v-btn @click = "view='form'" class="white--text pink">Adat szerkesztése</v-btn>
        <v-btn @click="ujadatok, view='list'" class="white--text blue">Mégse</v-btn>
      </div>
      <div class="c1" v-if="view=='list'">
        <v-layout row v-if="view=='list' && this.tomb.length">
          <v-text-field
              label = "Keresés / Szűrés"
              v-model = "keres"/>
          <v-btn
            v-if  = "keres"
            small
            @click = "keres=''"
            class = "red--text">
          <i class = "material-icons">
          clear
          </i>
          </v-btn>
        </v-layout>
		   <v-select
			  v-if="view=='list' && this.tomb.length"
			  v-model="kategoria"
			  :items="listview"
			  label="keresés kategória szerint">
		  </v-select>
		  <v-select
			  v-if="view=='list' && this.tomb.length"
			  v-model="sorbarendez"
			  :items="listview"
			  label="sorbarendezés kategória szerint">
		  </v-select>
		  <v-btn
			v-if="view=='list' && this.tomb.length"
			@click="ujadatok, view='ujelem'"
			title="új adat felvétele"
            fab dark small
            color="red">
            <v-icon
            dark>add</v-icon>
		  </v-btn>
		  <v-btn 
			v-if="view=='list' && this.tomb.length"
			@click="deleteall"
            title="adatbázis ürítése"
            fab dark small
            color="red">
		    <i class="material-icons">	  
			delete_forever
			 </i>
          </v-btn>
		  <br>
		  <br>
        <div xs-text-center align="center">
		<table v-if="this.tomb.length">
			<tr>
				<th>#</th>
				<th v-for="(elem, key) in listview">
				  <span class="fejrow">{{elem}}</span>
				</th>
				<th>
				  <span class="fejrow">Műveletek</span>
				</th>
			</tr>
			<tr v-for="(sor, kulcs) in lista" :key="kulcs">
				<td>
				  {{kulcs+1}}
				</td>
				<td v-for="(value, key) in sor">
				  <span>
					{{ value }}
				  </span>
				</td>
				<td>
				  <span
					small
					@click="edit(sor), view='reszl'"
					class = "g orange--text">
					<i class = "material-icons"
					title = "Részletek megjelenítése">
					dvr
				    </i>
				  </span>
				  <span
					small
					@click = "edit(sor), view='form'"
					class = "g green--text">
					<i class = "material-icons"
					title = "Szerkesztés, módosítás">
					{{editic}}
				  </i>
				  </span>
				  <span
					small
					class = "g red--text"
					@click = "töröl(sor)">
				    <i class = "material-icons"
					title = "Törlés">
					{{deleteic}}
				  </i>
				  </span>
				</td>
			  </tr>
		</table>
        </div>
      </div>
      <div class="c1" v-if="view=='form'">
          <div>
          <table class = "i1">
            <tr v-for="(value, key) in record">
              <td style="width:20%; test-align:left;">
                {{key}}
              </td>
              <td style="width:40%;">
                <v-text-field
                  class    = "ihalf"   
				  :value="value"
				  v-model="record[key]"
                />
              </td>
            </tr>
          </table>
          </div>
          <v-btn @click="ujadatok, view='list'" class="white--text orange">Mégse</v-btn>
            &nbsp;
          <v-btn @click="doEdit(record), view='list'" class="white--text green">Változtatások mentése</v-btn>
      </div>
	  <div class="c1" v-if="view=='ujelem'">
          <div>
          <table class = "i1">
            <tr v-for="(value, key) in record">
              <td style="width:20%; test-align:left;">
                {{key}}
              </td>
              <td style="width:40%;">
                <v-text-field
                  class    = "ihalf"   
				  :value="value"
				  v-model="record[key]"
                />
              </td>
            </tr>
          </table>
          </div>
          <v-btn @click="ujadatok, view='list'" class="white--text orange">Mégse</v-btn>
            &nbsp;
          <v-btn @click="ujelemmentese(record), view='list'" class="white--text green">Új adatok felvétele</v-btn>
      </div>
  </v-app>
</template>

<script>

const config = {
  kiírás: 'Példa nyilvántartás',
  editicon: 3,      //1..3
  deleteicon: 3,     //1..5
}
const editiconlist = ['edit', 'build', 'update']
const deleteiconlist = ['delete', 'backspace', 'delete_sweep', 'remove', 'remove_circle', 'cancel']

export default {
  data: () => ({
    view: 'list',
    listview: [],
	tomb: [],
    kiiras:   config.kiírás,
    editic:   editiconlist[config.editicon-1],
    deleteic: deleteiconlist[config.deleteicon-1],
	keres: '',
	loading: false,
	cache: {},
    record: {},
	kategoria: "",
	sorbarendez: ""
    }),
	
    watch: {

    },

    methods: {
	loadTextFromFile(ev) {
		this.listview=[]
		this.tomb=[]
		this.storagesetter()
		this.loading=true
        const file = ev.target.files[0];
		var reader = new FileReader();
		reader.onload = e => this.loadHandler(e);
		reader.onerror = e => this.errorHandler(e);
		reader.readAsText(file);
    },
	errorHandler(evt) {
		if(evt.target.error.name == "NotReadableError") {
			this.loading=false
			alert("Cannot read file!");
		}
	},
	loadHandler(event) {
		let csv = event.target.result;
		let allTextLines = csv.split(/\r\n|\n/);
		let keys = allTextLines.shift().split(';');
		keys.forEach(v => this.listview.push(v))
		this.kategoria=this.listview[0]
		this.sorbarendez=this.listview[0]
		while (allTextLines.length) {
			let arr = allTextLines.shift().split(';');
			let obj = new Object();
			for(var i = 0; i < this.listview.length; i++){
				obj[this.listview[i]] = arr[i];
			}
		this.tomb.push(obj);
		}
		this.ujadatok()
		this.storagesetter()
		this.loading=false
	},
    töröl(ezt) {
		this.tomb.splice(this.tomb.indexOf(ezt), 1);
		this.ujadatok()
		this.storagesetter()
		
    },
    deleteall() {
		this.tomb=[]
		this.listview=[]
		this.ujadatok()
		this.storagesetter()
		
    },
	ujadatok(){
		this.record = new Object
		this.cached = new Object
		for(var i = 0; i < this.listview.length; i++){
				this.record[this.listview[i]] = "";
				this.cached[this.listview[i]] = "";
			}
	},
	ujelemmentese(ujelem){
		this.tomb.push(ujelem)
		this.ujadatok()
		this.storagesetter()
	},
	doEdit(record) {
		var index = this.tomb.indexOf(this.cache);
		this.tomb.splice(index, 1, record);
		this.ujadatok()
		this.storagesetter()
    },
    edit(record) {
		this.record = {...record};
		this.cache = record;
    },
    storagesetter(){
		localStorage.setItem("fejlec", JSON.stringify(this.listview));
		localStorage.setItem("adatok", JSON.stringify(this.tomb));
    },
    storagegetter(){
		this.listview=JSON.parse(localStorage.getItem("fejlec"));
		this.tomb=JSON.parse(localStorage.getItem("adatok"));
    }
    },

    computed: {
    lista(){
		return this.tomb.filter( v => {return (new RegExp(this.keres,'i').test( v[this.kategoria] ))})
						.sort( (a,b) => a[this.sorbarendez] ? a[this.sorbarendez].localeCompare(b[this.sorbarendez]) : 1 )
			}
    },

    mounted() {
		this.storagegetter()
		this.kategoria=this.listview[0]
		this.sorbarendez=this.listview[0]
		this.ujadatok()
	}
}
</script>

<style>
.fejrow {
  font-size:16px;
  color: #154741;
  text-shadow: 1px 1px 4px #aaa;
}
.reszl {
  font-size: 15px;
}
span.g {
  margin-right:   4px;
  margin-left:    4px;
  cursor:         pointer;
  text-shadow: 1px 1px 4px #aaa;
}
span.g:hover {
  text-shadow: 1px 1px 1px black;
}
tr.row0 {
  background-color: rgba(180, 220, 220, 0.3);
  border-radius: 5px;
  box-shadow: 1px 1px 2px gray;
}
tr.row1 {
  background-color: rgba(220, 180, 220, 0.3);
  border-radius: 5px;
  box-shadow: 1px 1px 2px gray;
}
td,th {
  font-size: 13px;
  padding-left:10px;
  padding-right:10px;
  color: #256761;
}
.i1 {
  width:400px;
  margin: 0 auto;
}
.ihalf {
  width:100px;
  margin: 0 auto;
}
.c1 {
  text-align: center;
  box-shadow: 1px 1px 4px;
  border-radius: 12px;
  padding:20px;
  background-color: #efedde;
}
h1 {
  box-shadow: 1px 1px 4px;
  border-radius: 12px;
  padding:20px;
  margin-bottom: 23px;
  background-color: #dfedde;
}
#app {
  margin: 20px;
  text-align: center;
  height:90%;
  background-color: white;
}
.devtool {
  visibility: hidden; /* set visibility to visible to show dev tools */
  text-align:right;
  color:red;
  box-shadow: 1px 1px 4px gray;
  border-radius: 12px;
  padding:5px;
  margin-top:15px;
  background-color: #efedde;
}
.text-reader {
  position: relative;
  overflow: hidden;
  display: inline-block;
  /* Fancy button looking */
  border: 2px solid black;
  border-radius: 5px;
  padding: 8px 12px;
  cursor: pointer;
}
.text-reader input {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
  opacity: 0;
}
</style>