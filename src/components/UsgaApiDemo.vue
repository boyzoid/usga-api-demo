<template>
  <div>

    <div v-if="!token">
        <h1>Login</h1>
        <p>You actually need to login to use this API...sorry</p>
        <p class="error" v-if="error">
          {{ error }}
        </p>
        <p>
          <label for="username">Username </label>
          <input type="text" id="username" v-model="username"/>
        </p>
        <p>
          <label for="password">Password </label>
          <input type="password" id="password" v-model="password"/>
        </p>
        <button type="button" @click="login()">Login</button>
    </div>
    <div v-if="token">
      <h1>Golf Course Search</h1>
      <p class="error" v-if="error">
        {{ error }}
      </p>
      <p>
        <label for="courseSearch">Course Name </label>
        <input type="text" id="courseSearch" v-model="courseSearch"/> <button type="button" @click="search()">Search</button>
      </p>
      <div id="courses">
        <div class="course" v-for="c in sortedCourses" :key="c.CourseID">
          <div class="bold">{{ titleCase( c.FullName ) }}</div>
          <div>
            {{ titleCase( c.City ) }} {{ c.State }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UsgaApiDemo',
  data(){
    return {
      baseurl : 'https://api.ghin.com/api/v1/',
      token: '',
      username: '',
      password: '',
      user: {},
      error: '',
      courseSearch: '',
      courses: []
    }
  },
  computed:{
    sortedCourses(){
      return this.courses.slice().sort( function( a,b ){
        return a.FullName.toLowerCase() < b.FullName.toLowerCase() ? -1 : 1
      } )
    }
  },
  methods:{
    async login() {
     this.error = undefined;
     let loginResponse = await loginUser( this.username, this.password, this.baseurl );
     if( loginResponse.token ){
       this.token = loginResponse.token;
       this.user = loginResponse.user;
     }
     else{
      this.error = "There was a problem logging in."
     }

    },
    async search() {
      this.error = undefined;
      this.courses = [];
      let searchResult = await searchCourses( this.courseSearch, this.baseurl, this.token )
      if( searchResult.courses ){
        this.courses = searchResult.courses;
        if( this.courses.length === 0 ){
          this.error = 'No course matched your search'
        }
      }
      else{
        this.error = "There was a problem getting courses."
      }

    },
    titleCase(value){
    if( !value ){
      return ''
    }
    else{
      return value.toLowerCase().replace(/(?:^|\s|-)\S/g, x => x.toUpperCase());
    }

    }
  }
}

async function loginUser( username, password, url ){
  let data = {
    user:{
      "email" : username,
      "password" : password
    }
  }
  let resp = await fetch( url + 'users/login.json',{
        method: 'POST',
        headers: {
          'Content-Type' : 'application/json'
        },
        body: JSON.stringify( data )
    }
  )
  return resp.json();
}
async function searchCourses( search, url, token ){
  let resp = await fetch( url + '/courses/search.json?name=' + search + '&include_tee_sets=false' ,{
    method: 'GET',
    headers: {
      'Authorization' : 'Bearer ' + token
    }
  } )

  return resp.json();
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
a {
  color: #42b983;
}
.error{
  color: #9d3232;
  font-weight: bold;
}

.course{
  margin-bottom: 10px;
  width:35%;
  text-align: left;
  margin-left: auto;
  margin-right: auto;
  padding: 8px;
  border: 1px solid #555555;
  border-radius: 4px;
}
.bold{
  font-weight: bold;
}
#courses > div:nth-of-type( even ) {
  background: #e0e0e0;
}
</style>
