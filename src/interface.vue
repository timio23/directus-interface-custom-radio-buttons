<template>
	<v-notice v-if="!choices" type="warning">
		No choices configured
	</v-notice>
	<div
		v-else
		class="radio-icon-buttons"
		:style="{
			'--v-radio-color': color,
		}"
	>
		<input
			:value="value"
			:field="field"
			:collection="collection"
			type="hidden"
			@input="handleChange($event.target.value, field)"
		/>
		<button
			v-for="item in choices"
			:key="item.value"
			class="v-icon-radio block"
			type="button"
			:aria-pressed="isChecked(value, item.value) ? 'true' : 'false'"
			:disabled="disabled"
			:class="{ checked: isChecked(value, item.value), block }"
			@click="selectOption(item.value, field)"
		>
			<span class="label type-text">
        <v-icon v-if="choice.icon" :name="choice.icon" filled />
        <span v-else-if="choice.svg_icon" class="v-icon" v-html="choice.svg_icon"></span>
        <img v-else-if="choice.image" class="v-icon" :src="renderImage(choice.image)"/>
        <slot name="label">{{ choice.text }}</slot>
      </span>
		</button>
	</div>
</template>
<script>
import useDirectusToken from './use-directus-token';
export default {
	emits: ['input'],
	props: {
    field: String,
    collection: String,
    value: String,
    disabled: {
      type: Boolean,
      default: false,
    },
    choices: {
      type: Array,
      default: null,
    },
    width: {
      type: String,
      default: null,
    },
  },
	inject: ['api'],
  methods: {
    selectOption(value, field){
      if(field == this.field){
        this.$emit('input', value);
      }
    },
    isChecked(input, value){
      return input == value;
    },
    renderImage(file_id, modified_on = new Date().toISOString()){
      if(file_id === null) return;
      const { addTokenToURL } = useDirectusToken(this.api);
      return addTokenToURL(`/assets/${file_id}?width=42&height=42&fit=cover&cache-buster=${modified_on}`);
    },
    handleChange(value, field) {
      if(field == this.field){
        this.$emit('input', value);
      }
    },
  },
	mounted() {
		console.log(`BatchMode: ${this.batchMode}`);
		console.log(`Choices:`);
		console.log(this.choices);
	},
};
</script>

<style>
body {
	--v-icon-radio-color: var(--primary);
}
</style>

<style lang="scss" scoped>
.radio-icon-buttons {
	--columns: 5;
	display: grid;
	grid-gap: 12px 32px;
	grid-template-columns: repeat(var(--columns), 1fr);

	@media (max-width: 600px) {
		--columns: 3;
	}
}

.v-icon-radio {
	display: flex;
	font-size: 0;
	text-align: center;
    background-color: transparent;
	border: none;
	border-radius: 0;
	appearance: none;
	& .v-icon {
		--v-icon-color: var(--foreground-subdued);

		svg {
			width: 100%;
			height: 100%;
			fill: var(--foreground-subdued);
		}
	}
    &> .v-icon {
		position: absolute;
        display: none;
	}
    & .label {
        display: block;
        width: 100%;

        & .v-icon {
            display: block;
            margin: 0 auto 3px;
            width: 42px;
            height: 42px;
            border: 2px solid transparent;
            border-radius: 50%;
            padding: 7px;
            background: var(--background-input);
            box-shadow: 0 0 2px rgb(0 0 0 / 30%);
        }
    }
    &:not(.checked) {

        & .label {

            & .v-icon {
                border-color: transparent;
            }
        }
    }
	&:disabled {
		cursor: not-allowed;
		.label {
			color: var(--foreground-subdued);
		}
		.v-icon {
			--v-icon-color: var(--foreground-subdued);
		}
	}
	&.block {
		position: relative;
		width: 100%;
		height: auto;
		padding: 10px; // 14 - 4 (border)
		border: 2px solid var(--border-normal);
		border-radius: var(--border-radius);
        &:hover {
            border-color: var(--border-normal-alt);
        }
		&::before {
			position: absolute;
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;
			background-color: var(--background-subdued);
			border-radius: var(--border-radius);
			content: '';
		}
		.label {
			z-index: 1;
		}
	}
	&:not(:disabled):hover {
		.v-icon {
			--v-icon-color: var(--foreground-subdued);
		}
	}
	&:not(:disabled).checked {
		.v-icon {
			--v-icon-color: var(--v-icon-radio-color);

			svg {
				fill: var(--v-icon-radio-color);
			}
		}
		&.block {
			border-color: var(--v-icon-radio-color);
			.label {
				color: var(--v-icon-radio-color);
			}
			&::before {
				background-color: var(--v-icon-radio-color);
				opacity: 0.1;
			}
		}
	}
}
</style>
