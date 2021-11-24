<template>
  <default-field :field="field">
    <template slot="field">
      <div class="flex items-center">
        <select v-model="country"
                class="mr-3 form-control form-select"
                :class="errorClasses"
                @input="handleCountyChange"
                :readonly="field.readonly"
        >
          <option value="tr">Türkiye</option>
          <option value="other">Diğer</option>
        </select>

        <div v-if="country === 'other'" class="w-full">
          <input :id="`${field.name}-country-code`" v-model="formattedValue"
                 class="w-full form-control form-input form-input-bordered"
                 placeholder="Ülke Kodu Giriniz örn: +381"
                 :class="errorClasses"
                 :readonly="field.readonly"
          />
          <p class="help-text my-2 absolute" :class="{'error-text text-danger': hasError}">Ülke Koduyla beraber telefon
            girin. Örn: +381</p>
        </div>


        <input :id="field.name" type="tel" v-else
               class="w-full form-control form-input form-input-bordered"
               :class="errorClasses"
               :placeholder="placeholder"
               v-mask="mask"
               v-model="value"
               :readonly="field.readonly"
        />
      </div>


      <p v-if="hasError" class="help-text error-text my-2 text-danger">
        {{ firstError }}
      </p>
    </template>
  </default-field>
</template>

<script>
import {FormField, HandlesValidationErrors} from 'laravel-nova'

const defaultFormat = '# (###) ### ## ##';

export default {
  mixins: [FormField, HandlesValidationErrors],

  props: ['resourceName', 'resourceId', 'field'],

  data() {
    return {
      country: 'tr',
    }
  },

  computed: {
    placeholder() {
      let useMask = this.field.useMaskPlaceholder;
      let placeholder = this.field.placeholder;

      if (useMask) {
        return this.mask;
      } else if (placeholder) {
        return placeholder;
      }

      return this.field.name
    },
    mask() {
      if (this.field.format) {
        return this.field.format;
      }

      return defaultFormat;
    }
  },
  methods: {
    setInitialValue() {
      this.value = this.field.value || '';
      this.formattedValue = this.value;

      if (this.formattedValue.startsWith('+')) {
        this.country = 'other';
      }
    },

    fill(formData) {
      if (this.country === 'other') {
        this.value = this.formattedValue;

        if (!this.formattedValue.startsWith('+')) {
          this.value = null;
        }
      }

      formData.append(this.field.attribute, this.value || '')
    },

    handleChange(value) {
      this.value = value
    },

    handleCountyChange() {
      this.value = null;
      this.formattedValue = null;
    }
  },
}
</script>
