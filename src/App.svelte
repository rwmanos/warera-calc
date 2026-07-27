<script>
  // @ts-ignore
  import Switch from "./Switch.svelte";

  const gunAttacks = {
    green: { attack: 55, critChance: 8 },
    blue: { attack: 80, critChance: 12 },
    purple: { attack: 109, critChance: 19 },
    gold: { attack: 150, critChance: 30 },
  };

  const glovePrecisions = {
    green: 8,
    blue: 14,
    purple: 23,
    gold: 35,
  };

  const helmetCritChancePresets = {
    green: 25,
    blue: 45,
    purple: 80,
    gold: 100,
  };

  const ammoPresets = {
    green: 10,
    blue: 20,
    purple: 40,
  };

  /**
   * @param {number | string} value
   */
  function formatNumber(value) {
    return new Intl.NumberFormat("de-DE", {
      maximumFractionDigits: 0,
    }).format(Number(value));
  }

  let gunAttack = $state(gunAttacks.purple.attack);
  let gunCritChance = $state(gunAttacks.purple.critChance);
  let glovePrecision = $state(glovePrecisions.blue);
  let helmetCritChance = $state(helmetCritChancePresets.blue);
  let ammo = $state(ammoPresets.green);
  let militaryrank = $state(24.5);
  let battleBonus = $state(66.36);
  let buff = $state(true);
  let highlightedFields = $state({
    gunAttack: false,
    gunCritChance: false,
    glovePrecision: false,
    helmetCritChance: false,
    ammo: false,
  });
  let selectedPreset = $state({
    gunAttack: "purple",
    gunCritChance: "purple",
    glovePrecision: "blue",
    helmetCritChance: "blue",
    ammo: "green",
  });

  /** @type {{ [key: string]: { startingValue: number; stepValue: number; points: number } }} */
  let barsData = $state({
    Attack: { startingValue: 100, stepValue: 25, points: 6 },
    Precision: { startingValue: 50, stepValue: 5, points: 5 },
    "Crit. chance": { startingValue: 10, stepValue: 5, points: 6 },
    "Crit. damage": { startingValue: 100, stepValue: 20, points: 6 },
  });

  let total_attack_with_battle_bonus = $derived(
    (gunAttack +
      barsData.Attack.points * barsData.Attack.stepValue +
      barsData.Attack.startingValue) *
      ((1 + militaryrank / 100) *
        (1 + ammo / 100) *
        (1 + (buff ? 60 : 0) / 100)),
  );
  let total_attack = $derived(
    total_attack_with_battle_bonus * (1 + battleBonus / 100),
  );
  let total_precision = $derived(
    glovePrecision +
      barsData.Precision.points * barsData.Precision.stepValue +
      barsData.Precision.startingValue,
  );
  let total_crit_chance = $derived(
    gunCritChance +
      barsData["Crit. chance"].points * barsData["Crit. chance"].stepValue +
      barsData["Crit. chance"].startingValue,
  );
  let total_crit_damage = $derived(
    helmetCritChance +
      barsData["Crit. damage"].points * barsData["Crit. damage"].stepValue +
      barsData["Crit. damage"].startingValue,
  );
  let total_miss_perc = $derived(Math.max(0, 100 - total_precision));
  let total_miss = $derived((total_miss_perc * total_attack) / 2);
  let total_ontarget_perc = $derived(100 - total_miss_perc);
  let total_ontarget_normal = $derived(
    total_ontarget_perc * ((100 - total_crit_chance) / 100) * total_attack,
  );
  let total_ontarget_crit = $derived(
    ((total_ontarget_perc * total_crit_chance) / 100) *
      total_attack *
      (total_crit_damage / 100),
  );
  let sum_attack = $derived(
    total_miss + total_ontarget_normal + total_ontarget_crit,
  );
  /**
   * @param {"gunAttack" | "gunCritChance" | "glovePrecision" | "helmetCritChance" | "ammo"} field
   */
  function flashField(field) {
    highlightedFields[field] = true;
    highlightedFields = { ...highlightedFields };

    setTimeout(() => {
      highlightedFields[field] = false;
      highlightedFields = { ...highlightedFields };
    }, 3000);
  }

  /**
   * @param {"gunAttack" | "gunCritChance" | "glovePrecision" | "helmetCritChance"} field
   */
  function clearPresetSelection(field) {
    selectedPreset[field] = "";
  }

  /**
   * @param {string} key
   */
  function increase(key) {
    barsData[key].points = Math.min(10, barsData[key].points + 1);
    barsData = { ...barsData };
  }

  /**
   * @param {string} key
   */
  function decrease(key) {
    barsData[key].points = Math.max(0, barsData[key].points - 1);
    barsData = { ...barsData };
  }
</script>

<div class="container">
  <h1>Parameters</h1>

  <div class="inputs">
    <div class="stat-group">
      <div class="color-buttons">
        {#each Object.entries(gunAttacks) as [color, values]}
          <button
            class="color-btn"
            class:selected={selectedPreset.gunAttack === color}
            type="button"
            aria-label={color}
            onclick={() => {
              selectedPreset.gunAttack = color;
              gunAttack = values.attack;
              gunCritChance = values.critChance;
              clearPresetSelection("gunCritChance");
              flashField("gunAttack");
              flashField("gunCritChance");
            }}
            style={`background-color: ${color};`}
          ></button>
        {/each}
      </div>
      <label>
        Gun - Attack
        <input
          bind:value={gunAttack}
          type="number"
          class:highlighted={highlightedFields.gunAttack}
          oninput={() => {
            clearPresetSelection("gunAttack");
            clearPresetSelection("gunCritChance");
            flashField("gunAttack");
          }}
        />
      </label>
    </div>

    <div class="stat-group">
      <div class="color-buttons">
        {#each Object.entries(gunAttacks) as [color, values]}
          <button
            class="color-btn"
            class:selected={selectedPreset.gunCritChance === color}
            type="button"
            aria-label={color}
            onclick={() => {
              selectedPreset.gunCritChance = color;
              gunAttack = values.attack;
              gunCritChance = values.critChance;
              clearPresetSelection("gunAttack");
              flashField("gunAttack");
              flashField("gunCritChance");
            }}
            style={`background-color: ${color};`}
          ></button>
        {/each}
      </div>
      <label>
        Gun - Crit%
        <input
          bind:value={gunCritChance}
          type="number"
          class:highlighted={highlightedFields.gunCritChance}
          oninput={() => {
            clearPresetSelection("gunCritChance");
            clearPresetSelection("gunAttack");
            flashField("gunCritChance");
          }}
        />
      </label>
    </div>

    <div class="stat-group">
      <div class="color-buttons">
        {#each Object.entries(glovePrecisions) as [color, value]}
          <button
            class="color-btn"
            class:selected={selectedPreset.glovePrecision === color}
            type="button"
            aria-label={color}
            onclick={() => {
              selectedPreset.glovePrecision = color;
              glovePrecision = value;
              flashField("glovePrecision");
            }}
            style={`background-color: ${color};`}
          ></button>
        {/each}
      </div>
      <label>
        Glove - Preci%
        <input
          bind:value={glovePrecision}
          type="number"
          class:highlighted={highlightedFields.glovePrecision}
          oninput={() => {
            clearPresetSelection("glovePrecision");
            flashField("glovePrecision");
          }}
        />
      </label>
    </div>

    <div class="stat-group">
      <div class="color-buttons">
        {#each Object.entries(helmetCritChancePresets) as [color, value]}
          <button
            class="color-btn"
            class:selected={selectedPreset.helmetCritChance === color}
            type="button"
            aria-label={color}
            onclick={() => {
              selectedPreset.helmetCritChance = color;
              helmetCritChance = value;
              flashField("helmetCritChance");
            }}
            style={`background-color: ${color};`}
          ></button>
        {/each}
      </div>
      <label>
        Helmet - Cr. dmg%
        <input
          bind:value={helmetCritChance}
          type="number"
          class:highlighted={highlightedFields.helmetCritChance}
          oninput={() => {
            clearPresetSelection("helmetCritChance");
            flashField("helmetCritChance");
          }}
        />
      </label>
    </div>

    <div class="stat-group">
      <div class="color-buttons">
        {#each Object.entries(ammoPresets) as [color, value]}
          <button
            class="color-btn"
            class:selected={selectedPreset.ammo === color}
            type="button"
            aria-label={color}
            onclick={() => {
              selectedPreset.ammo = color;
              ammo = value;
              flashField("ammo");
            }}
            style={`background-color: ${color};`}
          ></button>
        {/each}
      </div>
      <!-- svelte-ignore a11y_label_has_associated_control -->
      <label>
        Ammo
        <span
          class="read-only-value"
          class:highlighted={highlightedFields.ammo}
        >
          {ammo}
        </span>
      </label>
    </div>

    <div class="full-width"></div>

    <div class="stat-group military-group">
      <label>
        Military Rank
        <input bind:value={militaryrank} type="number" />
      </label>
    </div>

    <div class="stat-group inline-field">
      <label>
        Battle Bonus
        <input bind:value={battleBonus} type="number" step="1" min="0" />
      </label>
    </div>
    <div class="stat-group inline-field">
      <label>
        Buff/Pill
        <Switch bind:value={buff} design="slider" />
      </label>
    </div>
  </div>

  {#each Object.entries(barsData) as [key, item]}
    <div class="bar_row">
      <label class="label" for={`slider-${key}`}>
        {key}
      </label>

      <span class="value"
        >{item.startingValue + item.points * item.stepValue}</span
      >
      <span class="points">{item.points}</span>

      <input
        class="slider"
        id={`slider-${key}`}
        type="range"
        min="0"
        max="10"
        step="1"
        bind:value={item.points}
      />

      <button class="btn" onclick={() => decrease(key)}> - </button>
      <button class="btn" onclick={() => increase(key)}> + </button>
    </div>
  {/each}
  <h1>Status</h1>
  <p>
    ⚔️ {total_attack_with_battle_bonus.toFixed(1)} 💥 {total_ontarget_perc}% 💫 {total_crit_chance}%
    🎯 {total_crit_damage}%
  </p>
  ⚔️ {total_attack.toFixed(1)} with battle bonus ({battleBonus}%)
  <h1>Calc - 100 hits</h1>
  <div class="calc-output">
    <div class="calc-row">
      <span class="calc-label">miss</span>
      <span class="calc-value">{total_miss_perc}%</span>
      <span class="calc-label"> </span>
      <span class="calc-value">{total_miss_perc}%</span>
      <span class="calc-equals"
        >{total_miss_perc} * {total_attack.toFixed(0)} / 2</span
      >
      <span class="calc-value">{formatNumber(total_miss)}</span>
    </div>
    <div class="calc-row">
      <span class="calc-label">on-target</span>
      <span class="calc-value">{total_ontarget_perc}%</span>
      <span class="calc-label">normal ({100 - total_crit_chance}%)</span>
      <span class="calc-value"
        >{(total_ontarget_perc * (100 - total_crit_chance)) / 100}%</span
      >
      <span class="calc-equals"
        >{(total_ontarget_perc * (100 - total_crit_chance)) / 100} * {total_attack.toFixed(
          0,
        )}</span
      >
      <span class="calc-value">{formatNumber(total_ontarget_normal)}</span>
    </div>
    <div class="calc-row">
      <span class="calc-label">on-target</span>
      <span class="calc-value"></span>
      <span class="calc-label">crit ({total_crit_chance}%)</span>
      <span class="calc-value"
        >{(total_ontarget_perc * total_crit_chance) / 100}%</span
      >
      <span class="calc-equals"
        >{(total_ontarget_perc * total_crit_chance) / 100} * {total_attack.toFixed(
          0,
        )} * {total_crit_damage / 100}</span
      >
      <span class="calc-value">{formatNumber(total_ontarget_crit)}</span>
    </div>
    <div class="calc-row">
      <span class="calc-label">total dmg</span>
      <span class="calc-value"></span>
      <span class="calc-value"></span>
      <span class="calc-value"></span>
      <span class="calc-value"></span>
      <span class="calc-value">{formatNumber(sum_attack)}</span>
    </div>
  </div>
</div>

<style>
  .container {
    max-width: 1000px;
    margin: 2rem auto;
    font-family: sans-serif;
  }

  .bar_row {
    max-width: 200px; /* or width: 120px; */
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .inputs {
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
    margin-bottom: 1.5rem;
    align-items: flex-start;
  }

  .stat-group {
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
    min-width: 140px;
    flex: 1 1 140px;
  }

  .military-group {
    min-width: 120px;
    flex: 0 1 120px;
  }

  .inline-field {
    min-width: 120px;
    flex: 0 1 140px;
  }

  /**
   *Linebreak - Make the last row of inputs take the full width
   */
  .full-width {
    flex-basis: 100%;
    height: 0;
  }

  .inputs label {
    display: flex;
    flex-direction: column;
    gap: 0.2rem;
    font-weight: 600;
  }

  .color-buttons {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .color-btn {
    width: 0.95rem;
    height: 0.95rem;
    border: 1px solid #fff;
    border-radius: 2px;
    box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.15);
    cursor: pointer;
    padding: 0;
  }

  .color-btn:hover {
    transform: scale(1.05);
  }

  .color-btn.selected {
    outline: 2px solid #14b8a6;
    outline-offset: 2px;
    transform: scale(1.08);
  }

  .inputs input {
    padding: 0.25rem;
    font: inherit;
  }

  .inputs input.highlighted,
  .read-only-value.highlighted {
    animation: pulse 3s ease;
  }

  .read-only-value {
    display: inline-flex;
    padding: 0.25rem 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    background: #f9fafb;
    font-weight: 600;
    font: inherit;
  }

  @keyframes pulse {
    0% {
      box-shadow: 0 0 0 0 rgba(20, 184, 166, 0.7);
    }
    50% {
      box-shadow: 0 0 0 6px rgba(20, 184, 166, 0.25);
    }
    100% {
      box-shadow: 0 0 0 0 rgba(20, 184, 166, 0);
    }
  }

  .label {
    width: 120px; /* constant space for all labels */
    flex-shrink: 0;
    font-weight: 600;
    text-align: left;
  }

  .points {
    width: 28px;
    text-align: center;
    flex-shrink: 0;
    font-weight: 700;
  }

  .value {
    width: 28px;
    text-align: center;
    flex-shrink: 0;
    color: #666;
    font-weight: 600;
  }

  .btn {
    min-width: 24px;
    height: 24px;
    cursor: pointer;
  }

  .calc-output {
    display: grid;
    gap: 0.35rem;
    max-width: 800px;
  }

  .calc-row {
    display: grid;
    grid-template-columns: 100px 90px 140px 50px 280px 85px;
    align-items: center;
    gap: 0.5rem;
    font-family: monospace;
  }

  .calc-label {
    font-weight: 700;
  }

  .calc-value {
    text-align: right;
  }

  .calc-equals {
    text-align: center;
    color: #666;
  }
</style>
