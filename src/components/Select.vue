<template>
  <div id="app">
    <div id="SchoolSelect">
        <br>
        <br>
        <br>       
        <multiselect ref="select" v-model="school" :options="schoolOptions" :multiple="false" :close-on-select="true" :clear-on-select="false" :preserve-search="false" placeholder="Pick your school" label="name" openDirection="top" track-by="name">
            <template slot="Select" slot-scope="{ options, search, isOpen }"><span class="multiselect__single" v-if="values.length &amp;&amp; !isOpen">{{ values.length }} options selected</span></template>
        </multiselect>
        <p>First, select your school.</p> 
    </div>
    <br>
    <transition name="fade">
    <div id="CourseSelect" v-if="school">
    <!-- <div id="CourseSelect">    -->
        <multiselect ref="select" v-model="selected" :options="options" :multiple="true" :close-on-select="false" :clear-on-select="false" :preserve-search="false" placeholder="Select a Course" label="label" track-by="name" openDirection="top" :max-height="150">
            <template slot="Select" slot-scope="{ options, search, isOpen }"><span class="multiselect__single" v-if="values.length &amp;&amp; !isOpen">{{ values.length }} options selected</span></template>
        </multiselect>
        <p>Great! Now select your classes. </p>
        <!-- <p>Select your classes. </p> -->
    </div>
    </transition>
    <transition name="fade">
    <div id="buttons" v-if="selected.length != 0">
        <br>
    <p>Nice! Now hit go to check out your prerequisite graph!</p>
    <sui-button class="ui fade animated button" @click="deselect">
      <sui-button-content visible>Reset</sui-button-content>
      <sui-button-content hidden>
        <sui-icon name="undo icon"/>
      </sui-button-content>
    </sui-button>
    <sui-button class="ui fade animated button" @click="makeGraph">
      <sui-button-content visible>Go!</sui-button-content>
      <sui-button-content hidden>
        <sui-icon name="checkmark icon" />
      </sui-button-content>
    </sui-button>
    </div>
    </transition>
  </div>
</template>

<script>

  // import vueSelect from 'vue-select'
  // import * as uvic_data from '@/assets/uvic_data.json'
  import * as ubc_data from '@/assets/ubc_data.json'
  import { mapActions } from 'vuex'
  import { mapGetters } from 'vuex'
  import Multiselect from 'vue-multiselect'
 
  var uvic_data;
  fetch(
        "https://uvic.kuali.co/api/v1/catalog/courses/5d9ccc4eab7506001ae4c225"
        ).then(r=>r.json()
        ).then(j=>{
            uvic_data = j;
            console.log("Fetched course data.");
            // console.log(uvic_data);
        });

  export default {
    name: 'Select',
    data () {
        return {
            selected: [],
            show_courses: false,
            schoolOptions: [{name: "University of Victoria"},{name: "University of British Columbia"}],
            schoolLookup: {
                "University of Victoria": uvic_data,
                "University of British Columbia": ubc_data
            },
            school: "",
            used_data: uvic_data
        }
    },
    computed: {
        
        options: function() {
            // console.log("School:")
            // console.log(this.school.name);
            
            let options = []
            // Wait until uvic_data loads
            while(uvic_data == undefined);

            for (var i in uvic_data){
                var long_name = uvic_data[i].__catalogCourseId + ": " + uvic_data[i].title
                // console.log("Pushing:");
                // console.log(uvic_data[i]);
                
                options.push({name: uvic_data[i].__catalogCourseId, label: long_name})
            }
            return options
            },
        ...mapGetters([
            'selectedCourses',
            'selectedSchool'
        ]),
        selectedSchool: function() {
            return this.selectedSchool()
        },
        
    }, 
    methods: {
        selectAll() {
            const select = this.$refs.select;
            select.options.forEach(option => {
                select.select(option);
            });
      
      // bug caused by onAfterSelect
            select.open = false
        },
        deselect() {
            this.selected = []
        },
        ...mapActions( {
            selectCourses: 'selectCourses',
            selectSchool: 'selectSchool'
        } ),
        makeGraph(){
            let result = this.selected.map(a => a.name)
            console.log("Result:"+ result)
            if(this.selected.length === 0){
                alert("Please select at least one course!")
            } else {
                this.selectCourses(result)
                this.selectSchool(this.school)
                this.$router.push('graph')
            }
        }
    },
    components: {
        Multiselect
    },
    watch: {
        school: function() {
            this.used_data = this.schoolLookup[this.school.name]
        }
    }
  }
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

<style scoped>
body {
    font-family: 'Open Sans', sans-serif;
}
#app {
  max-width: 35em;
  margin: 1em auto;
  font-family: 'Open Sans', sans-serif;
  font-size: 20px;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>