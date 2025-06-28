<template>
  <div>
    <h2>Динамическая форма с валидацией</h2>
    <form @submit.prevent="submitForm">
      <div v-for="(field, index) in formFields" :key="index" class="form-field">
        <label v-if="field.label">{{ field.label }}</label>
        <component
          :is="field.component"
          :class="['form-component', { 'form-component_is-invalid': field.error }]"
          v-bind="field.component !== 'select' && field.props"
          :value="field.value"
          @change="(e: Event) => handleFieldChange((e.target as HTMLInputElement).value, field)"
          @blur="validateField(field)"
        >
          <template v-if="field.component === 'select'">
            <option v-for="option in field.props.options" :key="option.value" :value="option.value">
              {{ option.text }}
            </option>
          </template>
        </component>
        <div v-if="field.error" class="error-message">{{ field.error }}</div>
      </div>
      <button type="submit" :disabled="!isFormValid">Регистрация</button>
    </form>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

interface FormField {
  component: 'input' | 'select'
  label: string
  value: string | boolean
  error: string
  props: { [key: string]: any }
  validation: {
    required: boolean
    [key: string]: any
  }
}

const formFields = ref<FormField[]>([
  {
    component: 'input',
    label: 'Login',
    value: '',
    error: '',
    props: {
      type: 'text',
      placeholder: 'Введите ваше логин',
    },
    validation: {
      required: true,
      minLength: 3,
      maxLength: 255,
      message: 'Длина логина должна быть от 3 до 255 символов',
    },
  },
  {
    component: 'input',
    label: 'Email',
    value: '',
    error: '',
    props: {
      type: 'email',
      placeholder: 'Введите ваш email',
    },
    validation: {
      required: false,
      pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      message: 'Введите валидный email',
    },
  },
  {
    component: 'input',
    label: 'Password',
    value: '',
    error: '',
    props: {
      type: 'password',
      placeholder: 'Введите ваш пароль',
    },
    validation: {
      required: true,
      minLength: 6,
      maxLength: 30,
      message: 'Длина пароля должна минимум 6 символов',
    },
  },
  {
    component: 'select',
    label: 'Role',
    value: '',
    error: '',
    props: {
      options: [
        { value: '', text: 'Выберите роль' },
        { value: 'admin', text: 'Админ' },
        { value: 'user', text: 'Пользователь' },
      ],
    },
    validation: {
      required: true,
      message: 'Пожалуйста, выберите роль',
    },
  },
  {
    component: 'input',
    label: 'Согласие с политикой конфиденциальности',
    value: false,
    error: '',
    props: {
      type: 'checkbox',
    },
    validation: {
      required: true,
    },
  },
])

const validateField = (field: FormField) => {
  field.error = ''

  if (field.validation) {
    if (field.validation.required && !field.value) {
      field.error = 'Это поле обязательно для заполнения'
      return false
    }

    if (
      field.validation.minLength &&
      typeof field.value === 'string' &&
      field.value.length < field.validation.minLength
    ) {
      field.error = field.validation.message || `Минимальная длина: ${field.validation.minLength}`
      return false
    }

    if (
      field.validation.maxLength &&
      typeof field.value === 'string' &&
      field.value.length > field.validation.maxLength
    ) {
      field.error = field.validation.message || `Максимальная длина: ${field.validation.maxLength}`
      return false
    }

    if (
      field.validation.pattern &&
      typeof field.value === 'string' &&
      !field.validation.pattern.test(field.value)
    ) {
      field.error = field.validation.message || 'Некорректный формат'
      return false
    }
  }

  return true
}

const validateForm = () => {
  let isValid = true
  formFields.value.forEach((field) => {
    if (!validateField(field)) {
      isValid = false
    }
  })
  return isValid
}

const handleFieldChange = (value: FormField['value'], field: FormField) => {
  if (field.props?.type === 'checkbox') {
    field.value = !field.value
  } else {
    field.value = value
  }

  if (field.error) {
    validateField(field)
  }
}

const isFormValid = computed(() => {
  return formFields.value.every((field) => !field.error)
})

const formData = computed(() => {
  return formFields.value.reduce((acc: { [key: string]: any }, field) => {
    acc[field.label || field.component] = field.value
    return acc
  }, {})
})

const submitForm = () => {
  if (validateForm()) {
    console.log('Форма валидна. Отправленные данные:', formData.value)
  } else {
    console.log('Форма содержит ошибки')
  }
}
</script>

<style lang="scss">
.form-field {
  margin-bottom: 1rem;

  label {
    display: block;
    margin-bottom: 0.5rem;
  }

  input[type='checkbox'] {
    width: fit-content;
  }
}

.form-component {
  padding: 0.5rem;
  width: 100%;
  max-width: 300px;
  border-radius: 3px;
  border: 1px solid black;

  &_is-invalid {
    border: 1px solid #ff4444;
  }
}

.error-message {
  color: #ff4444;
  font-size: 0.8rem;
  margin-top: 0.25rem;
}

button {
  padding: 0.5rem;

  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
}
</style>
