<script>
  import axios from "axios";
  import { scale } from "svelte/transition";

  let ehrBase = axios.create({
    baseURL: "http://localhost:8080/ehrbase/rest/ecis/v1",
  });
  let form;
  let loading = false;
  let data;

  let injections = [];

  let committed = "";
  const ctx = {
    composer_name: "Dr. Sidharth R",
    territory: "IN",
  };
  async function submit(e) {
    loading = true;
    const r = await form.post(e.detail);
    committed = r.data.compositionUid;
    loading = false;
  }

  function handleImageClick(e) {
    const image = e.target.getBoundingClientRect();
    const x = e.clientX - image.left;
    const y = e.clientY - image.top;
    injections = [...injections, { x, y }];
  }

  async function load() {
    const uuid = "b6142333-74fe-472c-ac30-c90719d1a1c2::1";
    const composition = await form.get(uuid);
    const orders = form.getStructured(
      composition,
      "boutolin_medication/medication_order/order"
    );
    console.log(orders);
    injections = orders.map((order) => ({
      x:
        order["anatomical_location"]["precise_anatomical_location"]["x_offset"][
          "magnitude"
        ],
      y:
        order["anatomical_location"]["precise_anatomical_location"]["y_offset"][
          "magnitude"
        ],
    }));
    setTimeout(() => {
      data = composition;
    });
  }
</script>

<section class="section">
  <div class="container">
    <h1 class="subtitle">Botulin Administration Form</h1>
    <mb-form
      bind:this={form}
      on:load={load}
      on:mb-input={(e) => (data = e.target.data)}
      {data}
      on:submit={submit}
      cdr={ehrBase}
      template="Boutolin medication"
      ehr="f0eaa2fd-e152-4dc2-afe8-9ed1674f8544"
      {ctx}
    >
      <mb-context path="boutolin_medication/category" />
      <mb-context path="boutolin_medication/context/start_time" />
      <mb-context path="boutolin_medication/context/setting" />
      <mb-context path="boutolin_medication/medication_order/order:0/timing" />
      <mb-context
        path="boutolin_medication/medication_order/order:0/action_archetype_id"
      />
      <mb-context path="boutolin_medication/medication_order/subject" />
      <mb-context
        path="boutolin_medication/medication_order/narrative"
        data="Giving injection?"
      />
      <mb-context path="boutolin_medication/medication_order/language" />
      <mb-context path="boutolin_medication/medication_order/encoding" />
      <mb-context path="boutolin_medication/composer" />
      <mb-context path="boutolin_medication/language" />
      <mb-context path="boutolin_medication/territory" />

      <div class="columns">
        <div class="column is-3">
          <svg width="180" height="250">
            <image
              on:click={handleImageClick}
              width="180"
              height="250"
              href="./me.jpg"
            />
            {#each injections as injection, i}
              <text x={injection.x + 3} y={injection.y - 10} fill="white"
                >{i + 1}</text
              >
              <circle cx={injection.x} cy={injection.y} r="5" fill="white" />
              <circle cx={injection.x} cy={injection.y} r="2" fill="black" />
            {/each}
          </svg>
        </div>
        <div class="column is-9 grid">
          {#each injections as injection, i}
            <mb-input
              class="is-hidden"
              data="Botulinum"
              path={`boutolin_medication/medication_order/order:${i}/medication_item`}
              label="Medication item"
            />
            <mb-input
              data={injection.x < 90 ? "Right face" : "Left face"}
              path={`boutolin_medication/medication_order/order:${i}/anatomical_location/body_site_name`}
              label="Body site name"
            />
            <mb-quantity
              class="is-hidden"
              data={{ unit: "1", magnitude: injection.x }}
              path={`boutolin_medication/medication_order/order:${i}/anatomical_location/precise_anatomical_location/x_offset`}
              label="X offset"
            >
              <mb-unit unit="1" label="1" />
            </mb-quantity>
            <mb-quantity
              class="is-hidden"
              data={{ unit: "1", magnitude: injection.y }}
              path={`boutolin_medication/medication_order/order:${i}/anatomical_location/precise_anatomical_location/y_offset`}
              label="Y offset"
            >
              <mb-unit unit="1" label="1" />
            </mb-quantity>
            <mb-quantity
              on:mb-input={(e) => {
                injection = { ...injection, amount: e.target.data };
              }}
              default="1"
              hideunit={true}
              path={`boutolin_medication/medication_order/order:${i}/therapeutic_direction/dosage/dose_amount`}
              label="Dose amount (mL)"
            >
              <mb-unit unit="1" label="1" />
            </mb-quantity>
            <p class="align-end">{injection.amount?.magnitude * 50} units</p>
          {/each}
          <div />
          <p class="has-text-weight-bold">
            Total: {injections.length
              ? injections
                  .map((i) => i.amount?.magnitude || 0)
                  .reduce((total, num) => total + num)
              : 0} mL
          </p>
          <p class="has-text-weight-bold">
            {injections.length
              ? injections
                  .map((i) => i.amount?.magnitude || 0)
                  .reduce((total, num) => total + num) * 50
              : 0} units
          </p>
          <div class="field">
            <mb-submit {loading} type="primary">Submit</mb-submit>
          </div>
        </div>
      </div>
      {#if committed}
        <div class="field" transition:scale>
          <sl-alert open={true} type="success">
            <sl-icon slot="icon" name="check2-circle" />
            <strong>Successfully committed composition</strong>
            <br />
            {committed}
          </sl-alert>
        </div>
      {/if}
    </mb-form>
  </div>
</section>

<style>
  .grid {
    display: grid;
    grid-template-columns: 2fr 2fr 1fr;
    align-self: end;
    column-gap: 1rem;
    row-gap: 1rem;
  }

  .align-end {
    align-self: end;
  }
</style>
