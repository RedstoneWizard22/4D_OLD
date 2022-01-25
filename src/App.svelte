<script lang="ts">
  import { Router, Route } from 'svelte-routing';
  import Projection from './routes/projection.svelte';
  import Slicing from './routes/slicing.svelte';
  import Home from './routes/home.svelte';

  export let url = '';
</script>

<main>
  <Router {url}>
    <Route path="projection" component={Projection} />
    <Route path="slicing" component={Slicing} />
    <Route path="/"><Home /></Route>
  </Router>
</main>
