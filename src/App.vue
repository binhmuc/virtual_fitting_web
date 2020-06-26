<template>
  <div id="app">
    <header>
      <router-link to="/"
                   class="route">Demo</router-link>
    </header>

    <router-view/>
  </div>
</template>

<script>
export default {
  name: "App"
};
</script>

<style>
#app {
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen,
    Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 16px;
  text-align: center;
  color: #2c3e50;
  
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.spacer {
  flex: 1;
}

header {
  padding: 0.5rem 0;
}

.route {
  color: #7f8c8d;
  padding: 0rem 1rem;
}

body {
  margin: 0;
  background-color: #eee;
}

@keyframes octocat-wave {
  0%,
  100% {
    transform: rotate(0);
  }
  20%,
  60% {
    transform: rotate(-25deg);
  }
  40%,
  80% {
    transform: rotate(10deg);
  }
}

footer {
  padding: 1rem;
  background-color: #efefef;
  text-align: center;
  color: #9fa9aa;
  font-weight: 200;
  font-size: 1rem;
}
</style>
