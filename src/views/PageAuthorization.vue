<template>
  <form @submit.prevent="submit()">
    <v-card-title class="justify-center">Авторизация</v-card-title>
    <v-alert v-if="errorSubmit.success === false" type="error">
      {{ errorSubmit.message }}
    </v-alert>
    <v-text-field
      color="teal"
      v-model="phone"
      :error-messages="phoneErrors"
      label="Телефон"
      @input="$v.phone.$touch()"
      @blur="$v.phone.$touch()"
      v-mask="'+7(###)###-##-##'"
      :counter="16"
    ></v-text-field>
    <v-text-field
      color="teal"
      :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
      :type="showPassword ? 'text' : 'password'"
      @click:append="showPassword = !showPassword"
      v-model="password"
      :error-messages="passwordErrors"
      label="Пароль"
      @input="$v.password.$touch()"
      @blur="$v.password.$touch()"
    ></v-text-field>
    <v-checkbox
      color="teal"
      v-model="checkbox"
      label="Запомнить пароль?"
    ></v-checkbox>
    <v-btn block color="teal" dark class="mb-2" type="submit">
      Войти
    </v-btn>
    <div class="text-center mt-4">
      <v-btn to="/forgotpassword" plain color="teal">
        Забыли пароль?
      </v-btn>
      <v-btn to="/registration" plain color="teal" dark>
        Регистрация
      </v-btn>
    </div>
  </form>
</template>

<script>
import { validationMixin } from "vuelidate";
import { required, minLength, maxLength } from "vuelidate/lib/validators";
export default {
  name: "PageAuthorization",
  mixins: [validationMixin],

  data: () => ({
    showPassword: false,
    phone: "",
    password: "",
    checkbox: false,
    errorSubmit: {}
  }),

  validations: {
    phone: { required, minLength: minLength(16), maxLength: maxLength(16) },
    password: { required, minLength: minLength(8) }
  },

  computed: {
    phoneErrors() {
      const errors = [];
      if (!this.$v.phone.$dirty) return errors;
      (!this.$v.phone.minLength || !this.$v.phone.maxLength) &&
        errors.push("Телефон введен некорректно");
      !this.$v.phone.required && errors.push("Телефон не указан");
      return errors;
    },
    passwordErrors() {
      const errors = [];
      if (!this.$v.password.$dirty) return errors;
      !this.$v.password.minLength &&
        errors.push("Пароль должен содержать не менее 8 символов");
      !this.$v.password.required && errors.push("Пароль не указан");
      return errors;
    }
  },

  methods: {
    submit() {
      this.errorSubmit = {};
      this.$v.$touch();

      if (this.$v.$invalid) {
        return;
      }

      const data = {
        phone: this.phone.replace(/[+()-]/g, ""),
        password: this.password
      };

      this.$store
        .dispatch("profile/login", data)
        .then(() => {
          localStorage.removeItem("phoneLogin");
          this.$router.push({
            name: "User"
          });
        })
        .catch(err => {
          this.errorSubmit = err.data;
        });
    }
  },
  mounted() {
    if (localStorage.getItem("phoneLogin")) {
      this.phone = localStorage.getItem("phoneLogin");
    }
  },
  watch: {
    phone(newPhone) {
      localStorage.setItem("phoneLogin", newPhone);
    }
  }
};
</script>
