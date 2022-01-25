<script>
  import { Link } from 'svelte-routing';
</script>

<div>
  <p class="text-center text-2xl py-3">Home</p>
  <div class="w-max mx-auto flex space-x-2">
    <Link to="projection">Projection</Link>
    <Link to="slicing">Slicing</Link>
  </div>
</div>
