<template>
  <div id="app">
    <nav class="navbar">
      <h1>English Listening Practice</h1>
    </nav>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.navbar {
  background-color: #42b983;
  padding: 1rem;
  color: white;
  margin-bottom: 2rem;
}

.navbar h1 {
  margin: 0;
  font-size: 1.5rem;
}
</style> 