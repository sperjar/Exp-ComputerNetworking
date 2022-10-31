<template>
  <nav text-xl mt-6 inline-flex gap-2>
    <router-link to="/">
      <div inline-block icon-btn i-carbon-campsite />
    </router-link>
    <button icon-btn @click="toggleDark()">
      <div inline-block dark:i-carbon-moon i-carbon-sun />
    </button>
    <a
      icon-btn i-carbon-logo-github rel="noreferrer" href="https://github.com/sperjar/Exp-ComputerNetworking"
      target="_blank" title="GitHub"
    />
  </nav>
</template>
