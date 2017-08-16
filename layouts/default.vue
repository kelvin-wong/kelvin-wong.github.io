<template>
  <div>
    <nav class="navbar">
      <nuxt-link active-class="is-active" class="nav-item" to="/" exact>Home</nuxt-link>
      <nuxt-link active-class="is-active" class="nav-item" to="/about" exact>About</nuxt-link>
      <nuxt-link active-class="is-active" class="nav-item" to="/timer" exact>Timer</nuxt-link>
    </nav>
    <section class="hero is-large">
      <div class="hero-body">
        <div class="container has-text-centered">
          <nuxt/>
        </div>
      </div>
    </section>
  </div>
</template>

<style>
html
{
  font-family: "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 16px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
}
*, *:before, *:after
{
  box-sizing: border-box;
  margin: 0;
}
.links
{
  padding-top: 15px;
}
nav
{
  min-height: 5vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  box-shadow: #DDD 0 2px 10px 0;
}
.nav-item
{
  text-decoration: none;
  justify-content: flex-start;
}
.nav-item.is-active
{
  color: red;
}
</style>
