<script lang="ts">
  import Form from "./lib/Form.svelte";
  import Header from "./lib/Header.svelte";
  import Pokemon from "./lib/Pokemon.svelte";
  import TableSup from "./lib/TableSup.svelte";
  import TableInf from "./lib/TableInf.svelte";
  import Errors from "./lib/Errors.svelte";

  type Pokemon = {
    id: number;
    name: string;
  };

  //   GET

  let page = 1;
  let error = false;
  let pokemonList: Pokemon[] = [];

  const getPokemon = async (page: number) => {
    const res = await fetch(
      `http://localhost:4321/api/pokemon.json?page=${page}`
    );
    const data = await res.json();
    pokemonList = data.pokemonList;
    return data;
  };

  $: error, page, getPokemon(page);

  async function handleNextPage() {
    const totalPages = Math.ceil(pokemonList.length / 5);

    if (page <= totalPages) {
      page += 1;
      console.log("Página siguiente:", page);
    }
  }

  function handlePrevPage() {
    page = Math.max(1, page - 1);
    console.log(page);
  }

  //   POST

  async function handleSubmit(e: { detail: { id: number; name: string } }) {
    const { id, name } = e.detail;
    const response = await fetch("http://localhost:4321/api/pokemon.json", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ id, name }),
    });
    if (response.ok) {
      pokemonList = [...pokemonList, { id, name }];
      console.log(pokemonList);
    }
  }

  // DELETE

  async function handleDelete(e: { detail: { pokemonId: number } }) {
    const { pokemonId } = e.detail;
    const response = await fetch(
      `http://localhost:4321/api/pokemon/${pokemonId}.json`,
      {
        method: "DELETE",
      }
    );
    if (response.ok) {
      pokemonList = pokemonList.filter((pokemon) => pokemon.id !== pokemonId);
    }
  }
</script>

<Header />
<main>
  <Form on:submit={handleSubmit} />
  <!-- {#if error}
  <div class="bg-red-500 text-white p-2 mb-4">El pokemon elegido ya existe</div>
  {/if} -->
  <div class="border-4 border-[#b6410f] rounded-xl bg-[#b6410f] mx-auto w-4/5">
    <TableSup />
    {#each pokemonList as item}
      <Pokemon
        pokemonId={item.id}
        pokemonName={item.name}
        on:delete={handleDelete}
      />
    {/each}
  </div>
  <TableInf on:nextPage={handleNextPage} on:prevPage={handlePrevPage} />
</main>
