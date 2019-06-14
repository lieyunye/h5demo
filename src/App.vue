<template>
  <Home/>
</template>

<script>
import Home from './components/home'
export default {
  name: 'app',
  components: {
    Home
  },
  mounted() {
    let performance = window.performance || window.msPerformance || window.webkitPerformance;
    if (performance && performance.timing) {
	    let t = performance.timing;
      let navigationStart = t.navigationStart; //跳转开始时间
      let pageDownLoadedTime = t.responseEnd - navigationStart;
      console.log('pageDownLoadedTime: ' + pageDownLoadedTime)
      let appStartTime = t.domContentLoadedEventStart;
      console.log('readyState: ' + document.readyState + new Date().getTime())
      console.log('window readyState: ' + window.readyState + new Date().getTime())
      window.onreadystatechange = () => {
        console.log('window readyState: ' + window.readyState + new Date().getTime())
      }
      window.addEventListener("load", function(event) {
        console.log("All resources finished loading!" + (new Date().getTime() - navigationStart));
      });
      document.onreadystatechange = () => { 
      console.log('readyState: ' + document.readyState + new Date().getTime())

      if (document.readyState == "complete") { 
          // run code here
      } 
  }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
