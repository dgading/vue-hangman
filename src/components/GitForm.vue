<template>
  <form>
    <label>Github Username:
      <input type="text" id="github_username"/>
    </label>
    <input type="submit" value="Get Repo Name">
  </form>
</template>

<script>
export default {
  name: "GitForm"
};
</script>

<style scoped>
input {
  font-family: "IBM Plex Mono", monospace;
}
</style>
