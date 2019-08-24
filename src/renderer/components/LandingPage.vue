<template>
  <div id="wrapper">
    <div class="cont">
      <div id="table">
        <b-input-group class="mt-3 mb-3" size="sm">
          <b-form inline @submit="run" action="#">
            <b-input
              id="inline-form-input-name"
              class="mb-2 mr-sm-2 mb-sm-0"
              placeholder="Search"
              v-model="keyword"
              @input="clear"
              autofocus
              ></b-input>

            <b-button type="submit" variant="outline-primary">Seach</b-button>
          </b-form>
        </b-input-group>
        <div class="row" v-if="search">
          <div class="col-lg-6 col-md-6 col-sm-6 col-xs-6">
            <b-table v-if="!grep" :fields="t_grep" :items="igrep" ></b-table>
            <v-progress-circular 
                     v-else 
                     :size="50"
                     color="primary"
                     indeterminate>
            </v-progress-circular>
          </div>
          <div class="col-lg-6 col-md-6 col-sm-6 col-xs-6">
            <b-table v-if="!find" :fields="t_find" :items="ifind" ></b-table>
            <v-progress-circular 
                     v-else 
                     :size="50"
                     color="primary"
                     indeterminate>
            </v-progress-circular>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import SystemInformation from './LandingPage/SystemInformation'

  const { ipcRenderer } = require('electron')
  let dir = ""

  ipcRenderer.on('ping', (event, msg) => {
    dir = msg
  })
  export default {
    name: 'landing-page',
    components: { SystemInformation },
    data () {
      return {
        keyword: '',
        grepArray: [],
        findArray: [],
        t_grep: [
          {key: 'row', label: 'Grep', sortable: true}
        ],
        t_find: [
          {key: 'row', label: 'Find', sortable: true}
        ],
        search: false,
        grep: false,
        find: false
      }
    },
    methods: {
      open (link) {
        this.$electron.shell.openExternal(link)
      },
      shell: function shell(cmd) {
        const exec = require('child_process').exec;
        return new Promise((resolve, reject) => {
          exec(cmd, {maxBuffer: 1024 * 1000}, (error, stdout, stderr) => {
            resolve(stdout? stdout : stderr);
          });
        });
      },
      run_script: function run_script(command, args, callback) {
        var child_process = require('child_process');

        return new Promise((resolve, reject) => {
          console.log("Starting Process.");
          var child = child_process.spawn(command, args);


          child.stdout.setEncoding('utf8');
          child.stdout.on('data', function(data) {
            console.log('stdout: ' + data);
            resolve(data);
          });
        });
      },
      run: function(event){
        event.preventDefault()
        this.search = true
        this.grep = true
        this.find = true
        let keyword = this.keyword

        this.run_find(keyword)
        this.run_grep(keyword)
      },
      run_grep: function(keyword){

      //this.run_script("ls", ["-l", "/home"], function(output, exit_code) {

      //  console.log("Process Finished.");
      //  console.log('closing code: ' + exit_code);
      //  console.log('Full output of script: ',output);
      //});
        
        //const grep = this.run_script("grep", ["-r", `${keyword}`, `${dir}/app ${dir}/db ${dir}/spec`]);
      const grep = this.shell(`grep -r ${keyword} ${dir}/app`);
      grep.then(data => { 
        this.grepArray = data.split("\n").map(str => {
          str = str.split(dir)[1]
          return JSON.parse(`{"row": "${str}"}`);
        })
        this.grep = false
      })

      },
      run_find: function(keyword){
        const find = this.shell(`find ${dir}/app ${dir}/db ${dir}/spec -name "*${keyword}*"`);

        find.then(data => { 
          this.findArray = data.split("\n").map(str => {
            str = str.split(dir)[1]
            return JSON.parse(`{"row": "${str}"}`)
          })
          this.find = false
        })
      },
      clear: function(){
        if(this.keyword == ""){
          this.grepArray = []
          this.findArray = []
        }
      }
    },
    computed: {
      igrep () {
        return this.grepArray
      },
      ifind () {
        return this.findArray
      }
    }
  }
</script>

<style>
#table {
	margin: 0 auto;
	width: 70%;
  margin-top: 90px;
	
	.input-group-text {
		padding: 0 .5em 0 .5em;
		
		.fa {
			font-size: 12px;
		}
	}
}
::-webkit-scrollbar {
    display: none;
}
</style>
