<template>
  <div class="home">
    <User></User>
  </div>
</template>

<script>
// @ is an alias to /src
import User from "@/components/User.vue";

export default {
  name: "home",
  components: {
    User
  }
};
</script>
