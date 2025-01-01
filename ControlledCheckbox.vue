<template>
    <div class="d-flex align-center">
      <!-- <v-checkbox
        :input-value="localState"
        @click.native.prevent="onChange"
        color="#23B1A9"
        :disabled="disabled"
        :hide-details="hideDetails"
        class="my-0 py-0"
      /> -->
  
      <label class="form-control">
        <input
        :checked="localState"
        type="checkbox"
        @click.prevent="onChange"
        color="#23B1A9"
        class="my-0 py-0 px-2 py-2 checkbox"
      />
      <template v-if="text">
        <span class="mr-6 text-subtitle-2 grey--text text--darken-2 font-weight-medium"> {{ text }} </span>
      </template>
      <template v-else>
        <slot />
      </template>
    </label>
  
    </div>
  </template>
  
  <script>
  export default {
    name: "GenericCheckbox",
    props: {
      value: {
        default: false,
        type: Boolean,
      },
      class: {
        default: "",
        type: String,
      },
      text: {
        default: "",
        type: String,
      },
      disabled: {
        default: false,
        type: Boolean,
      },
      hideDetails: {
        default: false,
        type: Boolean,
      },
    },
  
    data: () => ({
      localState: false
    }),
  
    methods: {
      onChange() {
        this.message = ''
        const isChecked = !this.localState;
        
        if (isChecked) {
          this.$emit("checked");
        }
        else {
          this.$emit("unchecked");
        }
        this.$emit("input", isChecked);
        
        setTimeout(() => {
          this.localState = isChecked;
        }, 1000);
      }
    },
  }
  </script>
  
  <style scoped>
  
  .checkbox {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    background-color: var(--form-background);
    margin: 0;
    font: inherit;
    color: currentColor;
    width: 18px;
    height: 18px;
    border: 0.09em solid currentColor;
    border-radius: 0.15em;
    transform: translateY(-0.075em);
    display: grid;
    place-content: center;
  }
  
  .form-control {
    display: grid;
    grid-template-columns: 1em auto;
    gap: 0.5em;
  }
  
  
  input[type=checkbox]:checked::before {
    transform: scale(1);
  }
  
  input[type=checkbox]::before {
    content: "";
    width: 0.65em;
    height: 0.65em;
    -webkit-clip-path: polygon(14% 44%, 0 65%, 50% 100%, 100% 16%, 80% 0%, 43% 62%);
    clip-path: polygon(14% 44%, 0 65%, 50% 100%, 100% 16%, 80% 0%, 43% 62%);
    transform: scale(0);
    transform-origin: bottom left;
    transition: 120ms transform ease-in-out;
    box-shadow: inset 1em 1em var(--form-control-color);
    background-color: CanvasText;
  }
  </style>